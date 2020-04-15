---
layout: post
title:  Redis 从文件中批量插入数据
date:   2020-04-15 17:59:54
categories: Redis
---

* content
{:toc}

本文参考：[这里](http://www.redis.cn/topics/batch-insert.html)

为了测试，需要初始化一个八万数据的 Hash，步骤如下：

### 数据

创建 data.txt 文件，每行就是一个 hmset 命令：

	hmset key field1 value1
	hmset key field2 value2
	……

ps：文件中最好不要有空行或者在行末有空格，否则会出现意想不到的问题

### 转码

	> unix2dos data.txt
	unix2dos: converting file data.txt to DOS format ...

unix2dos 命令的安装:

	yum install unix2dos

### 导入

	> cat data.txt | redis-cli 
	OK
	OK
	……

如果数据量很大，可以使用```--pipe```参数来启用pipe协议，它可以减少返回结果的输出、更快的执行指令：

	cat data.txt | redis-cli --pipe
	All data transferred. Waiting for the last reply...
	Last reply received from server.
	errors: 0, replies: 81920

