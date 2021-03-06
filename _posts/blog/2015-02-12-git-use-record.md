---
layout:     	post
title:      	Git使用记录
category: 		blog
description: 	使用git到github的问题记录
notebook: 		个人
tags: 			个人, Markdown, Sublime Text
---
## Git GUI 乱码问题

### 在Git GUI中查看UTF-8编码的文本文件时

乱码类似：锘夸腑鏂囨枃妗￡

解决方案：

在Bash提示符下输入：

	git config --global gui.encoding utf-8

注：通过上述设置，UTF-8编码的文本文件可以正常查看，但是GBK编码的文件将会乱码，所以还是没有从根本上解决问题。

可行的方法之一为：将所有文本文件的编码统一为UTF-8或GBK，然后设置相应的gui.encoding参数为utf-8或gbk。

### 在sublime text中使用git命令乱码

解决方案：

	{
		"git_command": "D:/SyncDriver/Git/bin/git.exe"
	}

git所在的目录不要有空格和中文，跟path配置的git区分开来；

### permission denied (publickey)
> 该问题是在 Git Bash进行操作生成密钥出现的

打开Git GUI 帮助`show ssh key `，`generate key` 默认生成，复制粘贴到github的ssh key中添加即可；（该默认生成到 `./userName/.ssh 目录下`,默认名称为`id_rsa`，可能跟这名称有关系，未具体验证；）

针对原先push使用https的方式,修改为使用ssh，需要到项目的 `.git`文件夹(默认隐藏)下修改config 中的 url 为ssh链接；

### sublimet text git commit 中文乱码

直接使用Git GUI 和Git Bash 使用 ` git commit -m '中文' `提交中文到github正常；

直接使用sublime text 的 quick commit 输入中文操作，也不会乱码;

经多次尝试，发现该是 Git.sublime-package 中的 commit.py 中的代码有问题，在控制台中有输出`UnicodeDecodeError: ‘gbk’ codec can’t decode bytes in position 2-3: illegal multibyte sequence`，该问题是由于编码未正确解码导致；

在 插件 sublime-text-git github的项目下有看到[修复的提交](https://github.com/kemayo/sublime-text-git/commit/c2e5a0fd9bf7676222e825830dabdd603842092c);对此修改commit.py代码,，并替换到Git.sublime-package包中，解决该问题；

修改前代码：

	with open(message_file.name, 'r') as fp:

修改后代码：

	import codecs

	with codecs.open(message_file.name, mode = 'r',encoding = 'utf-8') as fp:


以下配置未起到作用：

~~ 使用文本编辑器打开文件%GIT_HOME%\etc\gitconfig，将下面显示的三个选项的字符集修改成如下：

	[gui]
     encoding = utf-8
	[i18n]
     commitencoding = gbk
	[svn]
     pathnameencoding = gbk
~~

### Git Bask ls不能显示中文目录 

解决办法：在git的安装目录下，D:\Git\etc\git-completion.bash中增加一行： 

	alias ls='ls --show-control-chars --color=auto'

## 参考链接

* [Git跨平台中文乱码临时解决方案](http://blog.csdn.net/snowdream86/article/details/6891319)
* [彻底解决Git中文乱码问题](http://www.diguage.com/archives/26.html)
* [Git插件](https://github.com/kemayo/sublime-text-git)
* [SublimeGit(暂未使用)](https://sublimegit.net/)
* [Git一套简流](http://www.cnblogs.com/BeginMan/p/3591005.html)