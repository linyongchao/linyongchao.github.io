---
layout: post
title:  "pyExcelerator"
date:   2015-09-17 19:26:05
categories: Python
---

* content
{:toc}

## 简介

[官网](https://pypi.python.org/pypi/pyExcelerator/)有详细介绍。
简单来说，其

* 支持 Excel 97/2000/XP/2003
* 支持 xls2txt, xls2csv, xls2html
* 只依赖 Python 2.4+，不依赖 Excel
* 使用起来比较简单

## 安装

下载

	wget https://pypi.python.org/packages/source/p/pyExcelerator/pyexcelerator-0.6.4.1.tar.bz2#md5=478c79a74be39d27eee9bd1b3032dffe

解压

	tar jxvf pyexcelerator-0.6.4.1.tar.bz2
	cd pyexcelerator-0.6.4.1

安装

	#Ubuntu
	sudo python setup.py install

	#CentOS
	su root
	python setup.py install

测试

	python
	from pyExcelerator import *

## 使用

### 建表
创建一个具有一张空表（mysheet）的 Excel 文件（test.xls）

	#!/usr/bin/python
	# -*- coding: utf-8 -*-
	# python version 2.7.6

	from pyExcelerator import *

	#实例化 Workbook
	w=Workbook()
	#增加工作表 Worksheet
	ws=w.add_sheet('mysheet')
	#保存
	w.save('test.xls')

### 写入
提供的写入方法为：

	write(row,col,context,style)
	参数 （行，列，内容，样式）

无样式，例如：

	ws.write(0,0,u"姓名")

数字格式化，例如：

	#实例一个style对象
	style = XFStyle()
	#设置相应的格式
	style.num_format_str = '##0.00'
	ws.write(0,1,10, style)

字体样式，例如：

	EnStyle = XFStyle()
	#字体
	EnStyle.font.name = 'Times New Roman'
	#加粗
	EnStyle.font.bold = True
	#删除线
	EnStyle.font.struck_out = True
	#中空
	EnStyle.font.outline = True
	ws.write(0,2,u"Jerry",EnStyle)

### 合并单元格
提供的方法为：

	write_merge( r1, r2, c1, c2, context, style)

列内合并：

	ws.write_merge( 2, 4, 1, 1, u"合并(2,1)-(4,1)")

行内合并：

	ws.write_merge( 2, 2, 4, 5, u"合并(2,4)-(2,5)")

合并多行多列：

	ws.write_merge( 1, 4, 6, 7, u"合并(1,6)-(4,7)")

需要注意的是：不知道出于什么原因考虑，源代码中并不支持列内合并！
在 Cell.py 文件的 82 行有一个断言：

	def __init__(self, parent, col1, col2, xf_idx):
		assert col1 < col2, '%d < %d is false'%(col1, col2)
		self.__parent = parent
		self.__col1 = col1
		self.__col2 = col2
		self.__xf_idx = xf_idx

若需要，将断言注释掉即可支持列内合并

### 行高和列宽

	#行高
	ws.row(0).height = 2000
	ws.row(1).height = 5000

	#列宽
	ws.col(0).width = 2000
	ws.col(1).width = 5000

值得注意的是：相同数值的行高和列宽实际距离并不相同
行高2000 ≈ 2.6 cm
列宽2000 ≈ 1.1 cm
实际单位有待具体研究

### 写入公式

	ws.write(0,0,1)
	ws.write(0,1,10)
	ws.write(0,2,13)
	ws.write(0,3,Formula("SUM(A1:C1)"))

（未完待续）
