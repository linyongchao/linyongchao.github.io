---
layout: post
title:  Mac 下安装配置 SQLPlus
date:   2021-05-25 17:19:05
categories: Mac Oracle
---

* content
{:toc}

转载自[这里](https://www.cnblogs.com/zhuxiang1633/p/13933545.html)  
太喜欢这文章的风格了，一句废话都没有。。。

### 下载

[下载路径](https://www.oracle.com/database/technologies/instant-client/macos-intel-x86-downloads.html)

![img](https://linyongchao.github.io/static/img/oracle-client.png)

### 解压

将两个压缩包解压到同一个目录 instantclient 下

### 配置环境变量

	vim ~/.bash_profile

增加如下内容：

	export ORACLE_CLIENT=/Users/mac/opt/oracle/instantclient
	export PATH=$PATH:$ORACLE_CLIENT
	# mac下防止中文乱码
	export NLS_LANG="AMERICAN_AMERICA.UTF8"
	
最后：
	
	source ~/.bash_profile

### 配置数据库

创建文件：

	vim instantclient/network/admin/tnsnames.ora

写入内容：

	wuhu_jh =
	  (DESCRIPTION =
	    (ADDRESS = (PROTOCOL = TCP)(HOST = 172.18.247.52)(PORT = 1521))
	    (CONNECT_DATA =
	      (SERVER = DEDICATED)
	      (SERVICE_NAME = meter)
	    )
	  )
	  
### 连接

命令格式为：sqlplus username/password@ip/servicename

	sqlplus meter/meter@172.18.247.52/meter

### 历史命令

安装 rlwrap

	brew install rlwrap
	
配置：
  
	vim .bash_profile

添加如下内容：

	alias sqlplus='rlwrap sqlplus'
	
