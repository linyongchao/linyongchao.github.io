---
layout: post
title:  "CentOS 下安装 MySQL-python"
date:   2015-09-14 17:26:05
categories: CentOS MySQL-python
---

* content
{:toc}

## 原因

若允许 yum 则非常简单

	yum install MySQL-python -y

否则就需要源码安装，过程如下：

## 过程

下载

	wget http://download.sourceforge.net/sourceforge/mysql-python/MySQL-python-1.2.3.tar.gz

解压

	tar zxvf MySQL-python-1.2.3.tar.gz
	cd MySQL-python-1.2.3

修改 site.cfg ，把 mysql_config 那一行取消注释，并改为： 

	mysql_config = /usr/lib64/mysql/mysql_config  （根据自己mysql安装位置定义）

安装

	python setup.py build
	python setup.py install

## 测试

	python
	import MySQLdb

## 注意

需要先安装 setuptools ，详情请见[这里](http://linyongchao.github.io/2015/09/14/centos-setuptools/)
