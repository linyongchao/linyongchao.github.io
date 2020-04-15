---
layout: post
title:  "CentOS 下安装 pip"
date:   2015-09-14 14:37:05
categories: Linux Python
---

* content
{:toc}

## 原因

Python 下安装软件经常使用 pip 命令，但是 CentOS 下默认不带此命令。

## 过程

下载

	wget https://bootstrap.pypa.io/get-pip.py --no-check-certificate

安装

	python get-pip.py
