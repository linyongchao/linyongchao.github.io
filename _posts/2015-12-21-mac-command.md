---
layout: post
title:  Mac 的命令
date:   2015-12-21 00:45:05
categories: Mac
---

* content
{:toc}

Mac 虽说类似 Linux，但是有些命令还是有所区别的，本文就用来记录遇到的特殊命令。

## 查看端口占用

命令形式：sudo lsof -i :端口号
例如：

	➜  ~  sudo lsof -i :8080
	COMMAND   PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
	java    19718  lin   50u  IPv6 0x69cbe10b08770d2b      0t0  TCP *:http-alt (LISTEN)
	➜  ~
