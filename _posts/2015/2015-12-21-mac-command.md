---
layout: post
title:  Mac 的命令
date:   2015-12-21 00:45:05
categories: Mac
---

* content
{:toc}

Mac 虽说类似 Linux，但是有些命令还是有所区别的，本文就用来记录遇到的特殊命令。

### 查看端口占用

命令形式：sudo lsof -i :端口号

例如：

	➜  ~  sudo lsof -i :8080
	COMMAND   PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
	java    19718  lin   50u  IPv6 0x69cbe10b08770d2b      0t0  TCP *:http-alt (LISTEN)
	➜  ~

### TOP 排序

Linux

	CPU排序		P
	内存排序		M
	
Mac

	CPU排序		ocpu
	内存排序		ovsize
	
### 重建索引

方法见[这里](https://www.jianshu.com/p/d76dbc097521)

1. 关闭进程：```sudo mdutil -a -i off```
2. 重启进程：```sudo mdutil -a -i on``` （系统会重建索引列表）
3. 删除旧索引：```sudo rm -rf /.Spotlight-V100/*```

引申阅读见[这里](https://sspai.com/post/33089)：```让部分文件不被 Spotlight 搜到```

1. 在【系统偏好设置】-->【聚焦】中设置搜索的内容或者不在搜索范围内的内容
2. 给文件夹添加后缀```.noindex```



