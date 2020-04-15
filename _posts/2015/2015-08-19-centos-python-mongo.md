---
layout: post
title:  CentOS 下安装 PyMongo 模块
date:   2015-08-19 13:26:05
categories: Linux Python MongoDB
---

* content
{:toc}

## 环境

* 64 位 CentOS 6.5
* Python 2.6.6
* MongoDB 3.0.5

## 过程

下载

	wget https://pypi.python.org/packages/source/p/pymongo/pymongo-3.0.3.tar.gz

解压

	tar zxvf pymongo-3.0.3.tar.gz

安装

	cd pymongo-3.0.3
	python setup.py  install

## 测试

进入 Python 后，导入 pymongo ，不报错即可

	python
	>>> import pymongo
