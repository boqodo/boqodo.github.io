---
layout: post
title: Github博客搭建记录
description: 
	notebook: 个人
	tags:个人, Markdown, Sublime Text
category: blog
---

##github博客搭建

> 官方支持通过jekyll引擎生成静态html文件，生成规则和方式由配置文件控制，应该是生成相应的web静态文件存放到`_site`目录下，通过该目录访问读取html文件；
> 在github搭建博客的网文到处都是，有点云里雾里没理解具体的情况，准备先搭建起来再说，直接fork别人的博客，修改下相应的配置信息；

	1. 直接参考[使用Github Pages建独立博客](http://beiyuu.com/github-pages/) fork 其项目修改；
	2. 准备搭建jekyll ruby 环境，查看运行方式


###jekyll 环境搭建


####准备环境

[Ruby网站](http://rubyinstaller.org/downloads/)下载[Ruby 2.1.5](http://dl.bintray.com/oneclick/rubyinstaller/ruby-2.1.5-i386-mingw32.7z?direct)

解压到指定目录，并配置改目录到环境变量的path下；

下载[DevKit-mingw64-32-4.7.2-20130224-1151-sfx.exe](http://cdn.rubyinstaller.org/archives/devkits/DevKit-mingw64-32-4.7.2-20130224-1151-sfx.exe)
执行提取到指定目录，并配置该目录到环境变量path下；

进入DevKit的安装目录，打开命令窗口执行：

	$ruby dk.rb init

在目录下生成`config.yml`文件，打开该文件在文件最后添加：

	- {ruby安装路径}

最后执行安装：
	
	$ruby dk.rb install


####jekyll安装

安装jekyll执行:

	$gem install jekyll

使用rdiscount 作为markdown解释器(在`_config.yml`中配置)

	$gem install jekyll rdiscount

测试jekyll服务，进入博客目录执行：
	
	$jekyll serve --safe --watch

通过`localhost:4000`访问查看;

####原理验证
> 删除原先博客目录下的所有目录，只剩下运行jekyll测试环境时生成的_site目录，并上传到github看能否正常运行


###参考资料

- [使用Github Pages建独立博客](http://beiyuu.com/github-pages/)
- [一步步在GitHub上创建博客主页4](http://www.pchou.info/web-build/2013/01/05/build-github-blog-page-04.html)
- [使用Octopress搭建个人博客](http://sonnewilling.com/blog/2013/11/14/shi-yong-octopressda-jian-ge-ren-bo-ke/)
- [如何搭建一个独立博客——简明Github Pages与Hexo教程](http://cnfeat.com/2014/05/10/2014-05-11-how-to-build-a-blog/)
- [在Github上搭建Jekyll博客和创建主题](http://yansu.org/2014/02/12/how-to-deploy-a-blog-on-github-by-jekyll.html)
- [Github Pages/GitCafe pages 可以搭建博客，并且可以绑定域名，是基于什么原理的呢？](http://www.zhihu.com/question/26609475)
