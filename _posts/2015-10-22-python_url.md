---
layout: post
title:  "Python 添加自定义模块路径"
date:   2015-10-22 13:49:05
categories: Python
---

* content
{:toc}

## 原因

我们知道，自己写的 Python 程序属于自定义模块，要想正常调用需要将其添加到环境变量中。

## 步骤

### 查看现有变量

	>>> import sys
	>>> sys.path
	['', '/usr/lib/python2.7', '/usr/lib/python2.7/plat-x86_64-linux-gnu', '/usr/lib/python2.7/lib-tk', '/usr/lib/python2.7/lib-old', '/usr/lib/python2.7/lib-dynload', '/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages/PILcompat', '/usr/lib/python2.7/dist-packages/gtk-2.0', '/usr/lib/pymodules/python2.7', '/usr/lib/python2.7/dist-packages/ubuntu-sso-client']

### 临时添加变量

	>>> sys.path.append('/home/lin/work/git/celloud/python')

### 永久添加变量

打开文件：

	sudo vi /etc/profile

添加：

	export PYTHONPATH=$PYTHONPATH:/home/lin/work/git/celloud/python

重启之后再查看变量

	>>> import sys
	>>> sys.path
	['', '/home/lin', '/home/lin/work/git/celloud/python', '/usr/lib/python2.7', '/usr/lib/python2.7/plat-x86_64-linux-gnu', '/usr/lib/python2.7/lib-tk', '/usr/lib/python2.7/lib-old', '/usr/lib/python2.7/lib-dynload', '/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages/PILcompat', '/usr/lib/python2.7/dist-packages/gtk-2.0', '/usr/lib/pymodules/python2.7', '/usr/lib/python2.7/dist-packages/ubuntu-sso-client']

