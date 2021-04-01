---
layout: post
title:  RabbitMQ 安装
date:   2021-04-01 18:09:05
categories: MQ
---

* content
{:toc}

### 下载 RabbitMQ

在[这里](https://www.rabbitmq.com/install-generic-unix.html)下载安装包，目前版本是3.8.14  
解压之后，进入 sbin 目录，执行命令 ```./rabbitmq-server``` 启动服务，提示缺少 erl

	./rabbitmq-server: line 82: exec: erl: not found
	
官网也说：

	Make Sure Erlang/OTP is Installed
	This package requires a supported version of Erlang to be installed in order to run.
	
在[这里](https://www.rabbitmq.com/which-erlang.html)可以看到 RabbitMQ 3.8.14 依赖的 Erlang/OTP 版本  
RabbitMQ 3.8.14 依赖的 Erlang/OTP 版本为 22.3 ~ 23.x

### 安装 Erlang/OTP

根据[这里](https://www.erlang.org/downloads)安装 Erlang/OTP，命令

	brew install erlang
	
结果命令报错：

	Traceback (most recent call last):
		11: from /usr/local/Library/brew.rb:15:in `<main>'
		10: from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
		 9: from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
		 8: from /usr/local/Library/Homebrew/global.rb:7:in `<top (required)>'
		 7: from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
		 6: from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
		 5: from /usr/local/Library/Homebrew/os.rb:1:in `<top (required)>'
		 4: from /usr/local/Library/Homebrew/os.rb:16:in `<module:OS>'
		 3: from /usr/local/Library/Homebrew/os/mac.rb:16:in `version'
		 2: from /usr/local/Library/Homebrew/os/mac.rb:16:in `new'
		 1: from /usr/local/Library/Homebrew/os/mac/version.rb:25:in `initialize'
	/usr/local/Library/Homebrew/version.rb:186:in `initialize': Version value must be a string (TypeError)
	
根据[这里](https://www.88cto.com/article/ifOpB3m4)的提示，修改文件，写死版本：

	vi /usr/local/Library/Homebrew/version.rb
	
	def initialize(val)
		#raise TypeError, "Version value must be a string; got a #{val.class} (#{val})" unless val.respond_to?(:to_str)
		@version = '10.14.1'
	end

虽然文章里明确说了，```不要更新brew，不然代码可能会被覆盖掉。下次还得改一遍```，但我想着我从来没有更新过 brew 就更新一下吧，大不了更新完毕再改一次，结果报错了，执行命令```sudo brew update```，报错：

	Error: Cowardly refusing to `sudo brew update`
	You can use brew with sudo, but only if the brew executable is owned by root.
	However, this is both not recommended and completely unsupported so do so at
	your own risk.
	
根据[这里](https://blog.csdn.net/dongwuming/article/details/51691642)修改```/usr/local/bin/brew```的权限，从用户权限修改为root权限

	➜  ~ ll /usr/local/bin/brew
	-rwxr-xr-x  1 lin  admin   791B 10 30  2015 /usr/local/bin/brew
	
	➜  ~ sudo chown root /usr/local/bin/brew
	
	➜  ~ ll /usr/local/bin/brew
	-rwxr-xr-x  1 root  admin   791B 10 30  2015 /usr/local/bin/brew

修改权限后再执行```sudo brew update```，依然报错

	Error: Failure while executing: git stash pop --quiet
	
这个错误我以为是git连不上的问题，于是开启代理之后再执行```sudo brew update```，结果命令找不到了

	sudo: brew: command not found
	
根据[这里](https://blog.csdn.net/xiaolyuh123/article/details/106496162/)重新安装 brew

	/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
	
安装完毕之后觉得这个```sudo brew update```命令太诡异了，失败居然可以吧 brew 命令弄没了，而且这样安装应该是最新的，就决定不执行 update 了，直接安装```sudo brew install erlang```，结果依然报错

	Error: Running Homebrew as root is extremely dangerous and no longer supported.
	As Homebrew does not drop privileges on installation you would be giving all
	build scripts full access to your system.

看提示说不能用 root，去掉sudo试试```brew install erlang```，还是报错

	Updating Homebrew...
	Error: The following directories are not writable by your user:
	/usr/local/share/zsh
	/usr/local/share/zsh/site-functions
	
	You should change the ownership of these directories to your user.
	  sudo chown -R $(whoami) /usr/local/share/zsh /usr/local/share/zsh/site-functions
	
	And make sure that your user has write permission.
	  chmod u+w /usr/local/share/zsh /usr/local/share/zsh/site-functions
	  
看提示是权限问题，改下权限，从root改成了用户的

	➜  ~ ll /usr/local/share/zsh
	total 0
	drwxr-xr-x  3 root  wheel    96B  8 11  2017 site-functions
	
	➜  ~ sudo chown -R lin /usr/local/share/zsh /usr/local/share/zsh/site-functions
	Password:
	
	➜  ~ ll /usr/local/share/zsh
	total 0
	drwxr-xr-x  3 lin  wheel    96B  8 11  2017 site-functions
	
再执行一次 ```brew install erlang```，终于不报错了，执行 erl 看看

	➜  ~ erl
	Erlang/OTP 23 [erts-11.2] [source] [64-bit] [smp:8:8] [ds:8:8:10] [async-threads:1] [hipe] [dtrace]
	
	Eshell V11.2  (abort with ^G)
	1>
	
安装的版本是23，和下载的MQ是匹配的  

### 启动 RabbitMQ

erl 安装成功，就去RabbitMQ的sbin目录下执行```./rabbitmq-server```启动，这次正常了

	Configuring logger redirection
	
	  ##  ##      RabbitMQ 3.8.14
	  ##  ##
	  ##########  Copyright (c) 2007-2021 VMware, Inc. or its affiliates.
	  ######  ##
	  ##########  Licensed under the MPL 2.0. Website: https://rabbitmq.com
	
	  Doc guides: https://rabbitmq.com/documentation.html
	  Support:    https://rabbitmq.com/contact.html
	  Tutorials:  https://rabbitmq.com/getstarted.html
	  Monitoring: https://rabbitmq.com/monitoring.html
	
	  Logs: /Users/lin/Documents/rabbitmq_server-3.8.14/var/log/rabbitmq/rabbit@linycdeMacBook-Pro.log
	        /Users/lin/Documents/rabbitmq_server-3.8.14/var/log/rabbitmq/rabbit@linycdeMacBook-Pro_upgrade.log
	
	  Config file(s): (none)
	
	  Starting broker... completed with 0 plugins.
	  
### 安装监控插件

RabbitMQ 启动后，却无法访问```http://127.0.0.1:15672/```  
根据[这里](https://www.jianshu.com/p/60c358235705)安装可视化监控插件

	./rabbitmq-plugins enable rabbitmq_management
	
很顺利的安装了三个

	Enabling plugins on node rabbit@lin:
	rabbitmq_management
	The following plugins have been configured:
	  rabbitmq_management
	  rabbitmq_management_agent
	  rabbitmq_web_dispatch
	Applying plugin configuration to rabbit@linycdeMacBook-Pro...
	The following plugins have been enabled:
	  rabbitmq_management
	  rabbitmq_management_agent
	  rabbitmq_web_dispatch
	
	set 3 plugins.
	Offline change; changes will take effect at broker restart.
	
重启 RabbitMQ 后，```http://127.0.0.1:15672/```就可以访问了  
默认的用户名密码都是guest

### 配置环境变量

目前每次启动都要到 RabbitMQ 的 sbin 目录下执行```rabbitmq-server```  
根据[这里](https://www.jianshu.com/p/60c358235705)配置环境变量，配置完成之后可以直接使用```rabbitmq-server```命令了  
和文章中不同的是，我的环境变量是配置在```.bash_profile```文件中的

	export RABBIT_HOME=/Users/lin/Documents/rabbitmq_server-3.8.14
	export PATH=$PATH:$RABBIT_HOME/sbin
