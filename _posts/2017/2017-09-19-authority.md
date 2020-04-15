---
layout: post
title:  Linux权限
date:   2017-09-19 15:53:05
categories: Linux
---

* content
{:toc}

1. 文件权限: 
	* r: 可读，可以使用类似cat等命令查看文件内容
	* w: 可写，可以编辑或删除此文件
	* x: 可执行，eXacutable，可以命令提示符下当作命令提交给内核运行
2. 目录权限: 
	* r: 可以对此目录执行ls以列出内部的所有文件
	* w: 可以在此目录创建文件
	* x: 可以使用cd切换进此目录，也可以使用ls -l查看内部文件的详细信息
3. 权限三位一体: 
	* rwx: 可读可写可执行
	* r--: 只读
	* r-x: 读和执行
	* ---: 无权限
4. 八进制表示:  
	* 0 000 ---: 无权限
	* 1 001 --x: 执行
	* 2 010 -w-: 写
	* 3 011 -wx: 写和执行
	* 4 100 r--: 只读
	* 5 101 r-x: 读和执行
	* 6 110 rw-: 读写
	* 7 111 rwx: 读写执行
5. 例如: 
	* 755: rwxr-xr-x
	* 660: rw-rw----
	* rw-r-----: 640
	* rwxrwxr-x: 775		