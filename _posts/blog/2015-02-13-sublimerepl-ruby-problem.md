---
layout:     post
title:      SublimeREPL Ruby环境问题记录
category: blog
description: SublimeREPL Ruby环境问题记录
---

##SublimeREPL Ruby环境

###问题1:

```
ruby/1.9.1/rubygems/dependency.rb:247:in `to_specs': Could not find pry (>= 0) amongst [40] (Gem::LoadError)
    from /Users/ivaughan/.rbenv/versions/1.9.3-p194/lib/ruby/1.9.1/rubygems/dependency.rb:256:in `to_spec'
    from /Users/ivaughan/.rbenv/versions/1.9.3-p194/lib/ruby/1.9.1/rubygems.rb:1231:in `gem'
    from /Users/ivaughan/Library/Application Support/Sublime Text 2/Packages/SublimeREPL/config/Ruby/pry_repl.rb:2:in `<main>'

***Repl Closed***`
```
该问题是没有安装`pry`

	gem install pry


###问题2：

```
uninitialized constant Pry::InputCompleter (NameError)
``` 

> Digging into it I found that there is a bug in SublimeREPL that was causing Pry to not load/work correctly in ST2. Some suggested downgrading Pry which I did, but it just didn’t feel right. Then took a look at the pull request for the fix, found it wasn’t merged yet and followed it through to the file changes. 
I then opened up my local SublimeREPL /config/Ruby/pry_repl.rb file and made the changes and updated the file myself.

该问题是由于插件代码问题，修改pry_repl.rb文件，重启后即可；[具体修改参照](https://github.com/wuub/SublimeREPL/pull/372/files)


> 还有一种方式就是降级pry，见参考2



##参考

1. [Setting up SublimeREPL’s Pry Ruby CLI in Sublime Text 2 with bug workaround](http://vikingcodingadventures.tumblr.com/post/97746105649/setting-up-sublimerepls-pry-ruby-cli-in-sublime)
2. [Gem Pry - NameError: uninitialized constant Pry::BondCompleter::Bond](http://stackoverflow.com/questions/21405771/gem-pry-nameerror-uninitialized-constant-prybondcompleterbond)


