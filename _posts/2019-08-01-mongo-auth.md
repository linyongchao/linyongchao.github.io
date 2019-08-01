---
layout: post
title:  MongoDB集群增加认证
date:   2019-08-01 16:36:54
categories: MongoDB
---

* content
{:toc}

在搭建 MongoDB 集群的基础上，详情见[这里](https://linyongchao.github.io/2019/07/30/mongo/)，追加认证

### 生成密钥

在集群的任意一台服务器上执行下述命令，生成密钥：

	cd /usr/local/mongodb/
	mkdir key
	openssl rand -base64 756 > /usr/local/mongodb/key/key.file
	chmod 400 /usr/local/mongodb/key/key.file

将密钥复制到其他两台服务器

### 创建用户

在集群的任意一台服务器上执行下述命令，创建用户：

	cd /usr/local/mongodb/bin/
	./mongo --port 27017
	
	use admin
	db.createUser({user: "lin",pwd: "123456",roles: [ { role: "root", db: "admin" } ]})
	db.auth("lin","123456")

### 重启服务

三台服务器均执行

执行下述命令，杀死所有 MongoDB 进程：

	ps -ef|grep mongo|grep -v grep|awk '{print $2}'|xargs kill -9

然后执行下述命令，重启服务：

	nohup ./mongod -f /usr/local/mongodb/conf/config.conf --keyFile /usr/local/mongodb/key/key.file &
	./mongod -f /usr/local/mongodb/conf/shard1.conf --keyFile /usr/local/mongodb/key/key.file
	./mongod -f /usr/local/mongodb/conf/shard2.conf --keyFile /usr/local/mongodb/key/key.file
	./mongod -f /usr/local/mongodb/conf/shard3.conf --keyFile /usr/local/mongodb/key/key.file
	./mongos -f /usr/local/mongodb/conf/mongos.conf --keyFile /usr/local/mongodb/key/key.file

### 链接校验

    List<ServerAddress> addressList = new ArrayList<ServerAddress>();
    addressList.add(new ServerAddress("192.168.3.115", 27017));
    addressList.add(new ServerAddress("192.168.3.116", 27017));
    addressList.add(new ServerAddress("192.168.3.117", 27017));

    List<MongoCredential> credentialList = new ArrayList<MongoCredential>();
    MongoCredential credential = MongoCredential.createScramSha1Credential("lin", "admin", "123456".toCharArray());
    credentialList.add(credential);

    MongoClient mongoClient = new MongoClient(addressList, credentialList);
    MongoDatabase mongoDatabase = mongoClient.getDatabase("mytest");
    MongoCollection<Document> mongoCollection = mongoDatabase.getCollection("user");
    
    long count = mongoCollection.count();
    System.out.println(count);