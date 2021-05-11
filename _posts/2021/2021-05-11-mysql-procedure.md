---
layout: post
title:  MySQL 存储过程
date:   2021-05-11 18:39:05
categories: MySQL
---

* content
{:toc}

参考[这里](https://blog.csdn.net/chuangxin/article/details/83444879)

* PROCEDURE 存储过程

### 查看存储过程状态

	SHOW PROCEDURE STATUS LIKE 【name】;

### 查看存储过程定义

	SHOW CREATE PROCEDURE 【name】;

### 查看某张表涉及哪些存储过程

	SELECT * FROM mysql.proc WHERE body LIKE 【name】;
