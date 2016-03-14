---
layout: post
title:  "Python 判断字典是否包含某个 key"
date:   2016-03-04 16:57:05
categories: Python
---

* content
{:toc}

## has_key() 方法

该方法是为了支持2.2之前的代码留下的，不建议使用。Python3已删除该函数。

	>>> d = {'name':{},'age':{}}
	>>> print d.has_key('name')
	True
	>>> print d.has_key('sex')
	False

## 间接 in 方法

借助 d.keys() 列出字典所有的key

	>>> d = {'name':{},'age':{}}
	>>> print 'name' in d.keys()
	True
	>>> print 'sex' in d.keys()
	False
	>>>

## 直接 in 方法

官方文档推荐用法

	>>> d = {'name':{},'age':{}}
	>>> print 'name' in d
	True
	>>> print 'sex' in d
	False
	>>>

