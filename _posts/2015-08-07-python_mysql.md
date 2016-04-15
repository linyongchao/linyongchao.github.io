---
layout: post
title:  "python操作mysql"
date:   2015-08-07 12:04:54
categories: Python
---

* content
{:toc}

## 原因

最近在学习python，尝试使用其操作mysql数据库

---

## 配置类

新建 mysqlpro.py 作为配置类:

	#!/usr/bin/env python
	#encoding=utf-8
	#coding=utf-8

	__des__='mysql数据库配置文件'
	__author__='lin'

	class MySQLPro:
		host='localhost'
		user='root'
		password='root'
		db='test'

---

## 操作类

新建 mysqlOperate.py 作为数据库操作类:

	#!/usr/bin/env python
	#encoding=utf-8
	#coding=utf-8

	__des__='mysql数据库操作文件'
	__author__='lin'

	import sys
	import threading
	import MySQLdb
	from mysqlpro import MySQLPro

	class mysql:
		conn = None
		db = None
		cur = None
		instance = None

		locker = threading.Lock()

		@staticmethod
		def _connect():
			if not mysql.conn:
				#获取数据库连接
				mysql.conn=MySQLdb.connect(host=MySQLPro.host,user=MySQLPro.user,passwd=MySQLPro.password,charset='utf8')
				#选择数据库
				mysql.db=mysql.conn.select_db(MySQLPro.db)
				#获取操作游标
				mysql.cur=mysql.conn.cursor()

		#初始化
		def __init__(self):
			reload(sys)
			sys.setdefaultencoding('utf-8')
			mysql._connect()

		#获取单例对象
		@staticmethod
		def getInstance():
			mysql.locker.acquire()
			try:
				if not mysql.instance:
					mysql.instance = mysql()
				return mysql.instance
			finally:
				mysql.locker.release()

		#执行sql
		def execute(self,sql):
			try:
				#执行
				self.cur.execute(sql)
				#提交事物
				self.conn.commit()
			except MySQLdb.Error,e:
				print "Mysql Error %d: %s" % (e.args[0], e.args[1])
			finally:
				#关闭连接
				self.cur.close()
				self.conn.close()

---

## 测试类

新建 mysqltest.py 作为测试类:

	#!/usr/bin/env python
	#encoding=utf-8
	#coding=utf-8

	__des__='mysql数据库测试文件'
	__author__='lin'

	from mysqlOperate import mysql
	my=mysql.getInstance()
	if my:
		my.execute("add/update/delete ...")

---

## 小结

该操作类适合增删改操作，查询操作类似。
