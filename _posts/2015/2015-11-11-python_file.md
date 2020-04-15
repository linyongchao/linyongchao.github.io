---
layout: post
title:  Python 对文件和文件夹的相关操作
date:   2015-11-11 23:59:05
categories: Python
---

* content
{:toc}

Python 对文件、文件夹的操作通常涉及到 os 模块和 shutil 模块。

### 创建文件

	1. os.system('touch A') # 创建空文件 A
	2. open('A','w') # 打开 A 文件，如果不存在则创建

### 创建目录

	os.mkdir('A/B') # 创建目录 B，A 必须存在
	os.makedirs('A/B') # 创建目录 B，A 不存在则创建

### 复制文件

	1. shutil.copyfile('A','B') # A 和 B 都只能是文件
	2. shutil.copy('A','B') # A 只能是文件，B 可以是文件或目标目录

### 复制文件夹

	shutil.copytree('A','B') # A 和 B 都只能是目录，且 B 必须不存在

### 重命名

	os.rename('A','B') # 适用文件或目录

### 移动

	shutil.move('A','B') # 适用文件或目录   

### 删除文件

	os.remove('A')

### 删除目录

	os.rmdir('A') # 只能删除空目录
	shutil.rmtree('A') # 空目录、有内容的目录都可以 

### 切换目录

	os.chdir('A') # 相当于：cd A

### 判断

	os.path.exists('A') # 判断 A 是否存在
	os.path.isdir('A') # 判断 A 是否目录
	os.path.isfile('A') # 判断 A 是否文件

### 获取当前路径

	os.getcwd() # 相当于：pwd

### 遍历

	os.listdir('A') # 遍历 A 文件夹