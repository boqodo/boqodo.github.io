---
layout:     post
title:      Git使用记录
category: blog
description: 
---
##Git GUI 乱码问题

在Git GUI中查看UTF-8编码的文本文件时

乱码类似：锘夸腑鏂囨枃妗￡

解决方案：

在Bash提示符下输入：

	git config --global gui.encoding utf-8

注：通过上述设置，UTF-8编码的文本文件可以正常查看，但是GBK编码的文件将会乱码，所以还是没有从根本上解决问题。

可行的方法之一为：将所有文本文件的编码统一为UTF-8或GBK，然后设置相应的gui.encoding参数为utf-8或gbk。