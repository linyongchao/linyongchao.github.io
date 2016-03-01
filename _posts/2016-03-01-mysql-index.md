---
layout: post
title:  "MySQL 服务开始与停止"
date:   2015-09-14 19:57:05
categories: MySQL linux CentOS Ubuntu
---

* content
{:toc}

## CentOS

	service mysql restart
	/etc/init.d/mysqld start
	/etc/init.d/mysqld stop

## Ubuntu

	sudo /etc/init.d/mysql restart
	sudo /etc/init.d/mysql start
	sudo /etc/init.d/mysql stop
