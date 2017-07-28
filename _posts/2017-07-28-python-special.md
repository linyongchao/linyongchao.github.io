---
layout: post
title:  Python时间格式化
date:   2017-07-28 13:53:05
categories: Python
---

* content
{:toc}

## 数组插入

	A = ['aa','bb','cc']
	A.insert(0,'dd')
	A.insert(3,'ee')
	print A

	B = [['aa','bb']]
	B.insert(0,['aa','bb'])
	print B

## 一维数组转二维

	A = [u'123', u'abc']
	B = [ [A[i]] for i in range(len(A))]
	print B

## 三目

	a,b = 1,2
	print a if a > b else b
	print (a > b and [a] or [b])[0]