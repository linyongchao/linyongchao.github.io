---
layout: post
title:  Mac 下安装配置 SQL Plus、SQL Developer
date:   2021-05-25 18:39:05
categories: Mac Oracle
---

* content
{:toc}

## SQL Plus

转载自[这里](https://www.cnblogs.com/zhuxiang1633/p/13933545.html)  
太喜欢这文章的风格了，一句废话都没有。。。

### 下载

下载路径见[这里](https://www.oracle.com/database/technologies/instant-client/macos-intel-x86-downloads.html)

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
	
## SQL Developer

### 下载

下载路径见[这里](https://www.oracle.com/tools/downloads/sqldev-downloads.html)

### 配置

下载的文件解压后打开闪退  
根据[这里](https://blog.csdn.net/qq_38989725/article/details/114684337)配置 JDK 路径

1. 获取 JDK 路径：```echo $JAVA_HOME```
2. 打开配置文件：```vi ~/.sqldeveloper/20.4.1/product.conf```
3. 添加配置：```SetJavaHome 【JAVA_HOME】```

SQL Developer 就可以正常使用了
