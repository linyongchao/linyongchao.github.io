---
layout: post
title:  Oracle Command
date:   2018-04-27 16:25:54
categories: Oracle
---

* content
{:toc}

### 创建用户

1. 创建

		#创建用户 mytest ，密码 abc
		create user mytest identified by abc;
	
2. 授权

		grant connect,resource to mytest;
		grant create session to mytest;

3. 授空间

		alter user mytest quota 100m on users;

4. 切换用户

		connect mytest/abc;
		
ps:步骤3缺失，创建表时会报错：```ORA-01950: no privileges on tablespace 'USERS'```

### 切换用户

若不知道要切换用户的密码，可采用如下方式切换用户：

1. 以DBA身份连接数据库，创建一个代理用户并授予权限

		SQL> create user dbproxy identified by dbproxy;
		User created.
		SQL> grant connect to dbproxy;
		Grant succeeded.

2. 使目标用户可以通过代理用户切换

		SQL> alter user test grant connect through dbproxy;
		User altered.   

3. 登录测试

		SQL> conn dbproxy[test]/dbproxy
		Connected.
		SQL> show user
		USER is "TEST"
		SQL>

### 更改密码

	alter user sys indentified by test;
	
### 查看当前数据库连接用户

	show user