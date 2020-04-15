---
layout: post
title:  Python list
date:   2018-05-02 20:08:54
categories: Python
---

* content
{:toc}

Python 2.7.10 列表类型的相关操作  
[参考链接](http://www.runoob.com/python3/python3-data-structure.html)

### 基本操作

	a = ['a']
	b = ['b','c']
	a.append('d')		#追加
	a.extend(b)		#添加所有
	a.insert(3,'e')		#在指定位置插入 e
	a.remove('b')		#删除值为 b 的第一个元素
	c = a.pop(2)		#移除并返回索引位置元素
	d = a.index('c')	#返回 c 的索引
	e = a.count('d')	#统计 d 的次数
	a.sort()		#排序
	a.reverse()		#倒序
	
### 代码演示

	#!/usr/bin/python
	# -*- coding: utf-8 -*-
	
	a = ['a']
	b = ['b','c']
	
	#把一个元素添加到列表的结尾，相当于 a[len(a):] = [x]。
	a.append('d')
	print a
	
	#添加指定列表的所有元素，相当于 a[len(a):] = L。
	a.extend(b)
	print a
	
	#在指定位置插入一个元素
	a.insert(3,'e')
	print a
	
	#删除列表中值为 x 的第一个元素。如果没有这样的元素，就会返回一个错误。
	a.remove('b')
	print a
	
	#从列表的指定位置移除元素，并将其返回。如果没有指定索引，a.pop()返回最后一个元素。元素随即从列表中被移除。
	c = a.pop(2)
	print a	
	print c
	
	#返回列表中第一个值为 x 的元素的索引。如果没有匹配的元素就会返回一个错误。
	d = a.index('c')	
	print a
	print d
	
	#返回 x 在列表中出现的次数。
	e = a.count('d')	
	print a
	print e
	
	#对列表中的元素进行排序。
	a.sort()	
	print a
	
	#倒排列表中的元素。
	a.reverse()	
	print a


