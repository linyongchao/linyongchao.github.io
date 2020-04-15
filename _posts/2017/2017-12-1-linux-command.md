---
layout: post
title:  Linux Command
date:   2017-12-1 16:53:05
categories: Linux
---

* content
{:toc}

### 查看端口占用

	netstat -apn|grep [port]
	
### 修改所属用户、所属用户组

	chown -R root:root /tmp
	
### 筛选并截取

	cat login |grep 毫秒|awk -F 'controller' '{print $2}'
	
