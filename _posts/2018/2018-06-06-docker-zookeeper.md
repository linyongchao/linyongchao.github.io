---
layout: post
title:  Docker 搭建 ZooKeeper 集群
date:   2018-06-06 19:08:54
categories: Docker ZooKeeper
---

* content
{:toc}

学习 ZooKeeper 建议最少三个节点，所以尝试用 Docker 搭建 ZooKeeper 集群  
参考链接：[这里](https://segmentfault.com/a/1190000006907443)

### 镜像下载

	➜  ~ docker pull zookeeper
	Using default tag: latest
	latest: Pulling from library/zookeeper
	ff3a5c916c92: Pull complete
	a8906544047d: Pull complete
	a790ae7377b0: Pull complete
	2cbb132c648d: Pull complete
	5cb2cad05481: Pull complete
	d826b10121dc: Pull complete
	f05685c79e04: Pull complete
	Digest: sha256:96bd12586495a89da07d8bbd1fa48d127afe453adc428bb6a268ee2ec2ff7818
	Status: Downloaded newer image for zookeeper:latest

### 创建模板文件

进入目录```Documents/docker```，此后所有操作都在该目录下  
创建文件```zookeeper-compose.yaml```，内容如下：

	version: '2'
	services:
	    zoo1:
	        image: zookeeper
	        restart: always
	        container_name: zoo1
	        ports:
	            - "2181:2181"
	        environment:
	            ZOO_MY_ID: 1
	            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
	
	    zoo2:
	        image: zookeeper
	        restart: always
	        container_name: zoo2
	        ports:
	            - "2182:2181"
	        environment:
	            ZOO_MY_ID: 2
	            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
	
	    zoo3:
	        image: zookeeper
	        restart: always
	        container_name: zoo3
	        ports:
	            - "2183:2181"
	        environment:
	            ZOO_MY_ID: 3
	            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

### 启动镜像

	➜  docker docker-compose -f zookeeper-compose.yaml up
	Creating network "docker_default" with the default driver
	Creating zoo2 ... done
	Creating zoo1 ... done
	Creating zoo3 ... done
	Attaching to zoo2, zoo1, zoo3
	……
	后面很长的日志，不贴了
	
### 查看容器

新开一窗口，进入```Documents/docker```

	➜  docker docker-compose -f zookeeper-compose.yaml ps
	Name              Command               State                     Ports
	------------------------------------------------------------------------------------------
	zoo1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2181->2181/tcp, 2888/tcp, 3888/tcp
	zoo2   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2182->2181/tcp, 2888/tcp, 3888/tcp
	zoo3   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2183->2181/tcp, 2888/tcp, 3888/tcp
	
### 查看集群

	➜  docker echo stat | nc 127.0.0.1 2181
	Zookeeper version: 3.4.12-e5259e437540f349646870ea94dc2658c4e44b3b, built on 03/27/2018 03:55 GMT
	Clients:
	 /172.18.0.1:47398[0](queued=0,recved=1,sent=0)
	
	Latency min/avg/max: 0/0/0
	Received: 1
	Sent: 0
	Connections: 1
	Outstanding: 0
	Zxid: 0x0
	Mode: follower
	Node count: 4
	➜  docker echo stat | nc 127.0.0.1 2182
	Zookeeper version: 3.4.12-e5259e437540f349646870ea94dc2658c4e44b3b, built on 03/27/2018 03:55 GMT
	Clients:
	 /172.18.0.1:37092[0](queued=0,recved=1,sent=0)
	
	Latency min/avg/max: 0/0/0
	Received: 1
	Sent: 0
	Connections: 1
	Outstanding: 0
	Zxid: 0x0
	Mode: follower
	Node count: 4
	➜  docker echo stat | nc 127.0.0.1 2183
	Zookeeper version: 3.4.12-e5259e437540f349646870ea94dc2658c4e44b3b, built on 03/27/2018 03:55 GMT
	Clients:
	 /172.18.0.1:50862[0](queued=0,recved=1,sent=0)
	
	Latency min/avg/max: 0/0/0
	Received: 1
	Sent: 0
	Connections: 1
	Outstanding: 0
	Zxid: 0x100000000
	Mode: leader
	Node count: 4
	
### 关闭集群

	➜  docker docker-compose -f zookeeper-compose.yaml stop
	Stopping zoo3 ... done
	Stopping zoo1 ... done
	Stopping zoo2 ... done
	
### 总结

利用 Docker 搭建 ZooKeeper 集群非常简单，总体来说就是使用```docker-compose```命令