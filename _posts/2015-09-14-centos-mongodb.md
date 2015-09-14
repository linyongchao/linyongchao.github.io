---
layout: post
title:  "CentOS 下安装 MongoDB"
date:   2015-09-14 15:26:05
categories: CentOS MongoDB
---

* content
{:toc}

## 环境

* 64 位 CentOS 6.5

## 过程

下载

	wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel62-3.0.6.tgz

解压

	tar zxvf mongodb-linux-x86_64-rhel62-3.0.6.tgz

安装到/usr/local

	mv mongodb-linux-x86_64-rhel62-3.0.6 /usr/local/mongodb
	cd /usr/local/mongodb

配置 MongoDB

	mkdir db
	mkdir logs
	cd bin
	vi mongodb.conf

配置内容为

	dbpath=/usr/local/mongodb/db
	logpath=/usr/local/mongodb/logs/mongodb.log
	port=27017
	fork=true
	nohttpinterface=true

配置开机自动启动

	vi /etc/rc.d/rc.local

在文件最后新增一行

	/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf

## 测试

重启系统测试能否自动启动，然后：

	#进入mongodb的shell模式 
	/usr/local/mongodb/bin/mongo
	#查看数据库列表 
	show dbs
	#当前db版本 
	db.version();
