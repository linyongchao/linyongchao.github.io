---
layout: post
title:  "Ubuntu 打包 ISO 镜像"
date:   2016-02-27 19:57:05
categories: Ubuntu
---

* content
{:toc}

## 工具

Ubuntu 打包 ISO 镜像通常使用 remastersys 工具，点击[这里](https://github.com/mutse/remastersys)跳转其官网。

## 安装


	sudo add-apt-repository ppa:mutse-young/remastersys
	sudo apt-get update
	sudo apt-get install remastersys remastersys-gtk


## 使用


	sudo remastersys dist cdfs
	sudo remastersys dist iso custom.iso



