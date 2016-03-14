---
layout: post
title:  "Python 判断字典是否包含某个 key"
date:   2016-03-14 16:57:05
categories: Python
---

* content
{:toc}

## 方法一

has_key() 方法

	>>> d = {'name':{},'age':{}}
	>>> print d.has_key('name')
	True
	>>> print d.has_key('sex')
	False

## 方法二

in 方法，其中 d.keys() 列出字典所有的key

	>>> d = {'name':{},'age':{}}
	>>> print 'name' in d.keys()
	True
	>>> print 'sex' in d.keys()
	False
	>>>


