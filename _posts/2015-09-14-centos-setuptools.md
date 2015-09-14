---
layout: post
title:  "CentOS 下安装 setuptools"
date:   2015-09-14 17:36:05
categories: CentOS
---

* content
{:toc}

## 过程

下载

	wget http://pypi.python.org/packages/source/s/setuptools/setuptools-0.6c11.tar.gz

解压

	tar zxvf setuptools-0.6c11.tar.gz
	cd setuptools-0.6c11

安装

	python setup.py build
	python setup.py install
