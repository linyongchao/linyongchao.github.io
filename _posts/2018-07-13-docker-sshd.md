---
layout: post
title:  Docker 安装 SSH 并配置免密登录
date:   2018-07-13 19:06:54
categories: Docker
---

* content
{:toc}

### 简介

Docker 容器内安装 SSH 服务可以方便运维，但是是否要安装是有争议的  
我们暂且抛弃争议，只是安装并配置服务

### 准备

基础镜像 Ubuntu

	➜  ~ docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
	ubuntu              latest              113a43faa138        5 weeks ago          81.2MB

### 创建容器

	docker create -it -p 10022:22 ubuntu:latest
	f0f285e2aa……
	docker start f0f285e2aa
	docker exec -it f0f285e2aa /bin/bash
	
### 安装服务

安装了三个服务，ssh、netstat、vim

	apt-get update
	apt-get install openssh-server -y
	apt-get install net-tools
	apt-get install vim-gtk
	
其中 vim-gtk 安装过程中需要选择时区，我选择的亚洲、上海

	6. Asia
	69. Shanghai
	
### 启动服务

使用命令```netstat -tunlp```可以发现安装完毕的 ssh 并没有启动  
启动前需要创建文件夹：

	mkdir -p /var/run/sshd

然后启动：

	/usr/sbin/sshd -D &
	
再查看可以发现 ssh 服务已经启动

	root@f0a6ef9323b3:/# netstat -tunlp
	Active Internet connections (only servers)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      3863/sshd
	tcp6       0      0 :::22                   :::*                    LISTEN      3863/sshd

### 配置免密

	mkdir root/.ssh
	vi /root/.ssh/authorized_keys
	
authorized_keys 内的内容为本地```.ssh/id_rsa.pub```的内容

### 配置开机启动服务

创建文件```/run.sh```，内容如下：

	#!/bin/bash
	/usr/sbin/sshd -D

授予可执行权限：

	chmod +x run.sh
	
### 清理系统

由于基础镜像很干净，这一步可以省略，不过个人习惯还是会执行一下：

	apt-get autoclean	--清理旧版本的软件缓存
	apt-get clean		--清理所有软件缓存
	apt-get autoremove	--删除系统不再使用的孤立软件
	
### 制作镜像

制作镜像命令如下：

	docker commit f0f285e2aa sshd:ubuntu
	
制作成功，可以查看：

	➜  ~ docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
	sshd                ubuntu              19a091dd3303        About a minute ago   397MB
	
### 使用镜像

基于镜像启动容器：

	docker run -p 10022:22 -d sshd:ubuntu /run.sh
	
免密登录：

	➜  ~ ssh root@192.168.3.73 -p 10022
	Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.9.87-linuxkit-aufs x86_64)
	
	 * Documentation:  https://help.ubuntu.com
	 * Management:     https://landscape.canonical.com
	 * Support:        https://ubuntu.com/advantage
	
	This system has been minimized by removing packages and content that are
	not required on a system that users do not log into.
	
	To restore this content, you can run the 'unminimize' command.
	Last login: Fri Jul 13 18:31:32 2018 from 172.17.0.1
	root@3a0f0de8f0b8:~#