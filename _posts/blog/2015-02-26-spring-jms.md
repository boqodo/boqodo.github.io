---
layout: post
title: Spring集成JMS实践
description: 
category: blog
---

1. 参照`《Spring攻略(第二版)》`中的例子完成基本功能的实现

2. 使用以上完成的例子，修改为在`Webphere MQ`上测试通过

3. JMS事务，保持`数据一致性`的测试和研究

---
## JMS

### IBM MQ(WebSphere MQ)

- JMS API 实现收发消息
    + 获取ConnectionFactory
    + 创建连接cf.createConnection()
    + 创建会话conn.createSession()
    + 创建消息生产者session.createProducer(destination)
    + 创建消息session.createMapMessage()
    + 发送消息producer.send(message)

- JMS集成Spring实现收发消息[参考](http://docs.spring.io/spring/docs/3.0.x/spring-framework-reference/html/jms.html)
    + 使用JmsTemplate 处理收发(注入或继承JmsGatewaySupport获取)
    + 使用MessageConverter进行消息转换
    + 使用JmsListener 消息监听
    + 使用@Transactional控制事务处理(需要配置过`<tx:annotation-driven/>`)

- JMS消息监听处理
    + 需要在applicationContext配置文件中配置 监听器容器
       - SimpleMessageListenerContainer    不支持事务
       - DefaultMessageListenerContainer   支持事务
    + 消息驱动POJO,编写接收方法，通过适配器进行监听，接收到数据通过反射调入POJO指定的方法；
       - MessageListenerAdapter 适配器
    
    ```xml
     <bean id="jmsMessageListener" class="foo.bar.spring.JmsMessageListener"/>
     <bean id="listenerContainer" class="org.springframework.jms.listener.SimpleMessageListenerContainer">
       <property name="connectionFactory" ref="jmsConnectionFactory"/>
       <property name="messageListener" ref="jmsMessageListener"/>
       <property name="destinationName" value="${queue.name}"/>
     </bean>
    ```
   
- Spring 消息转换器 默认实现
 
    - SimpleMessageConverter
      基本的map、text、byte等的转换器
    - MarshallingMessageConverter
      使用JAXB的xml转换器
    - MappingJacksonMessageConverter
      使用Jackson json库的json和对象相互转换器

- Spring集成jms事务控制同jdbc的事务控制
    + 如何处理jdbc和jms事务在一个方法中[参考1](http://www.javaworld.com/article/2077963/open-source-tools/distributed-transactions-in-spring--with-and-without-xa.html)

**问题记录**

1. `com.ibm.mq.MQMsg2.getMessageData(Z)[B`错误提示;
    
    mq的jar同当前MQ服务器的版本不一致,复制websphere MQ安装目录`java`下的jar文件替换；
    需要jar包:
        + com.ibm.mq.jar
        + com.ibm.mq.jmqi.jar
        + com.ibm.mqjms.jar
        + dhbcore.jar

2. `javax.jms.JMSException: MQJMS2005: 未能为 '192.168.1.104:FirstLocalQ' 创建 MQQueueManager`错误提示;
        
		MQQueueConnectionFactory cf = new MQQueueConnectionFactory();
	    cf.setCCSID(1381);//必须要设置;一般都为1381,具体的值可以查看队列管理器属性中有指定

3. `java.lang.UnsatisfiedLinkError: no mqjbnd05 in java.library.path`错误提示;
        
		MQQueueConnectionFactory cf = new MQQueueConnectionFactory();
        cf.setTransportType(WMQConstants.WMQ_CLIENT_NONJMS_MQ);//需要设置为`WMQConstants.WMQ_CLIENT_NONJMS_MQ`相当于1
        
4. `DetailedJMSSecurityException: JMSWMQ2013: 为队列管理器 'FirstLocalQ' 提供的安全性认证无效`错误提示;
        
		MQQueueConnectionFactory cf = new MQQueueConnectionFactory();
        QueueConnection conn = null;
        conn = cf.createQueueConnection("MUSR_MQADMIN", "");//"MUSR_MQADMIN"为用户名,该用户名应该是默认通道`SYSTEM.DEF.SVRCONN`的默认用户名

5. 针对`上面一条(第4条)`的问题，在spring中如何配置用户名和密码呢?

    `MQQueueConnectionFactory`中无法通过属性注入用户名和密码，需要通过适配器(`UserCredentialsConnectionFactoryAdapter`)以中转的方式注入用户名和密码;
    
    对此在`JMSTemplate`和`JmsTransactionManager`中的connectionFactory引用需要是`适配器bean`;
    
    ```xml
     <!--WebSphere MQ Connection Factory 用户名和密码注入适配器-->
    <bean id="jmsConnectionFactory" class="org.springframework.jms.connection.UserCredentialsConnectionFactoryAdapter">
        <property name="targetConnectionFactory" ref="mqConnectionFactory"/>
        <property name="username" value="${app.mq.username}"/>
        <property name="password" value="${app.mq.password}"/>
    </bean>

    <!-- JMS Queue Connection Factory -->
    <bean id="transactionManager" class="org.springframework.jms.connection.JmsTransactionManager">
        <property name="connectionFactory" ref="jmsConnectionFactory"></property>
    </bean>

    <!-- Spring JmsTemplate -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="jmsConnectionFactory"/>
        <property name="receiveTimeout" value="${queue.receive.timeout}"/>
        <property name="defaultDestinationName" value="${queue.name}"/>
        <property name="messageConverter" ref="mapMessageConverter"/>
        <!--<property name="defaultDestination" ref="fqueue"/>-->
    </bean>
    <bean id="mapMessageConverter" class="foo.bar.spring.MapMessageConverter"></bean>
    <!-- JmsGatewaySupport -->
    <bean id="jmsSpring" class="foo.bar.spring.JmsSpring">
        <property name="connectionFactory" ref="jmsConnectionFactory"/>
        <property name="jmsTemplate" ref="jmsTemplate"/>
    </bean>
    <bean id="jmsMessageListener" class="foo.bar.spring.JmsMessageListener"/>
    <!-- 监听消息适配器 -->
    <bean id="jmsListenerAapdter" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
        <property name="delegate" ref="jmsReceiver"/>
        <!-- 可设置消息转换器-->
    </bean>
    <!-- 监听消息容器-->
    <bean id="listenerContainer" class="org.springframework.jms.listener.SimpleMessageListenerContainer">
        <property name="connectionFactory" ref="jmsConnectionFactory"/>
        <property name="messageListener" ref="jmsListenerAapdter"/>
        <property name="destinationName" value="${queue.name}"/>
    </bean>
    ```
6. `InvalidDestinationException`的异常提示;
    
    该异常的发生原因是使用`com.ibm.disthub2.impl.jms.MapMessageImpl`和`setStringProperty`方法造成的，该方法是配置消息头的额外属性，导致目标队列无效，而MQ服务器已经有数据，不过并没有响应的值，需要注意;


### Demo

[SpringJMSDemo](../files/SpringJmsDemo.rar)

### 参考资料

* [spring监听与IBM MQ JMS整合](http://blog.csdn.net/xiazou/article/details/19559247)
* [jdbc和jms同事务](http://stackoverflow.com/questions/9229573/spring-transaction-synchonization-of-jdbc-and-jms)
* [JMS and JDBC operations in one transaction with Spring/Jencks/ActiveMQ](http://activemq.apache.org/jms-and-jdbc-operations-in-one-transaction.html)
* [SPRING整合IBMMQ实现全局事物](http://www.blogjava.net/csusky/archive/2008/10/27/236912.html)
* [解读《使用Jencks实现Hibernate与Jackrabbit的分布式事务》](http://zhongl.iteye.com/blog/317041)

