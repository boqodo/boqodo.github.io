---
layout: post
title: RxJava 编程
category: blog
description: |-
  学习Rx相关知识点
  如何编写并且实战
---
Rx是一个`编程模型`，目标是提供`一致的编程接口`，帮助开发者更方便的处理`异步数据流`；


## [RxJava][1]

使用步骤：

1. create Observable 创建可观察对象
2. subscribeOn 指定订阅线程[可忽略，默认处理]
3. map、filter、catch 变换、过滤、错误处理
4. observeOn 指定观察者线程 [可忽略，默认处理]
5. subscribe 订阅


代码编写：

代码都是泛型操作，具体可以查看IDE代码提示方法的返回类型（Observable< R >），
确定传入的Funcx的具体泛型类；

变换、过滤、错误处理等参数基本都是 Funcx，根据返回的类型，确定具体的泛型；


### 创建操作
> 创建可观察对象(Observable)

- create
- just   
- from

### 变换操作
> 根据逻辑需要对Observable实现变换操作，针对旧的Observable携带的数据，进行逻辑处理后，返回一个新的Observable


- flatmap 无序

    将旧的Observable中的数据项，变换后，返回一个新的Observable对象

- map

    将旧的Observable中的数据项，变换后，返回一个新的数据项

- concatMap 有序

    同flatmap，该变换是有序的


### 过滤操作
> 根据一定的过滤条件过滤发射的数据项，将符合要求的数据项继续发射执行后续操作；

- filter
- first
- last

### 错误处理
> 在任何一层操作中出现异常，则都会在最后的 `subscribe` 的 `onError` 中处理；
>  如在相应的操作后添加错误处理，则根据错误处理的方式执行，继续后续操作；

- onErrorReturn
- onErrorResumeNext
- onExceptionResumeNext


## [RxAndroid][2]

### 新增线程控制  

- AndroidSchedulers.mainThread()
- AndroidSchedulers.handlerThread(handler)

### [Fragment和Activity生命周期][3]

需要针对Subscription 做unsubscribe 处理；

```java
// MyActivity
private Subscription subscription;

protected void onCreate(Bundle savedInstanceState) {
    this.subscription = observable.subscribe(this);
}

...

protected void onDestroy() {
    this.subscription.unsubscribe();
    super.onDestroy();
}
```
## 示例

[todo-mvp-rxjava](https://github.com/googlesamples/android-architecture/tree/todo-mvp-rxjava)

## 问题

1. 变换操作后是全部数据项转换完成后，转成一个新的发射序列再发射，还是转换一个就发射一个到下一个操作？

    转换一个发射一个，不过转换过程中可能转换成多个数据项发射；

2. 变换、过滤、错误处理等操作的参数基本都是Funcx的原因？

    只有一个Funcx的方法，是针对包含onNext、onError和onCompleted参数的方法的重载，
    Funcx对应的是onNext的参数；

## 参考

1. [ReactiveX RxJava文档中文版](https://www.gitbook.com/book/mcxiaoke/rxdocs/details)
2. [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
3. [给 Android 开发者的 RxJava 详解 - 印象笔记](https://app.yinxiang.com/shard/s6/nl/156766/e37b7039-a791-4d69-b009-61ee837cdd5e/)



[1]: https://github.com/ReactiveX/RxJava "RxJava"
[2]: https://github.com/ReactiveX/RxAndroid "RxAndroid"
[3]: https://mcxiaoke.gitbooks.io/rxdocs/content/topics/The-RxJava-Android-Module.html "Android应用"
