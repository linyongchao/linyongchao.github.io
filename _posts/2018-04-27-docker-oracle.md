---
layout: post
title:  Docker 安装 Oracle
date:   2018-04-27 17:39:54
categories: Docker Oracle
---

* content
{:toc}

1. 查看 Docker 版本

		docker --version

2. 检索 Oracle

		docker search oracle
		
		➜  ~ docker search oracle
		NAME                                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
		oraclelinux                         Official Docker builds of Oracle Linux.         446                 [OK]
		frolvlad/alpine-oraclejdk8          The smallest Docker image with OracleJDK 8 (…   301                                     [OK]
		sath89/oracle-12c                   Oracle Standard Edition 12c Release 1 with d…   285                                     [OK]
		alexeiled/docker-oracle-xe-11g      This is a working (hopefully) Oracle XE 11.2…   249                                     [OK]
		……

3. 下载 Oracle

		docker pull oraclelinux
		docker pull sath89/oracle-12c

4. 查看已下载镜像

		docker images

		➜  ~ docker images
		REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
		oraclelinux         latest              af8cf7fc5b7e        8 days ago          234MB
		sath89/oracle-12c   latest              17cd1ab9d9a7        4 months ago        5.7GB

5. 启动镜像

		sudo docker run -d -p 8088:8080 -p 1521:1521  sath89/oracle-12c
		
		-p 映射端口（宿主机端口：容器内部端口）

6. 查看镜像日志

		sudo docker logs -f ade82973e8cb

7. 服务

		docker ps
		
		不加参数，表示查看当前正在运行的容器 
		-a，查看所有容器包括停止状态的容器 
		-l，查看最新创建的容器 
		-n=x，查看最后创建的x个容器
		
		s  ~ docker ps
		CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                                  NAMES
		ade82973e8cb        sath89/oracle-12c   "/entrypoint.sh "   18 minutes ago      Up 18 minutes       1521/tcp, 8080/tcp, 0.0.0.0:8088-8089->8088-8089/tcp   zealous_kirch
			
		CONTAINER ID：容器ID，唯一标识容器 
		IMAGE：创建容器时所用的镜像 
		COMMAND：在容器最后运行的命令 
		CREATED：容器创建的时间 
		STATUS：容器的状态（你会看到UPXXX，表示运行状态） 
		PORTS：对外开放的端口号 
		NAMES：容器名（也具有唯一性，docker是不允许创建容器名相同的容器的）

8. 进入容器命令行

		docker exec -it ade82973e8cb	/bin/bash
		
9. 切换用户

		su oracle

10. 进入数据库

		$ORACLE_HOME/bin/sqlplus / as sysdba
		
	然后就是具体的 Oracle 数据库操作了，该数据库的其他信息如下
		
		hostname: localhost
		port: 1521
		sid: xe
		username: system
		password: oracle

11. 关闭镜像

		docker stop ade82973e8cb

