---
layout: post
title:  "tomcat_url"
date:   2015-08-06 16:54:54
categories: tomcat
---

* content
{:toc}

## 原因

某次用户请求的url总长度为12364字符,tomcat直接报400错误

---

## 分析

我之前一直知道一个"真理":

	get请求是限制长度的,而post请求是不限制长度的

在查询解决方案的过程中发现,上面的"真理"有问题:

	在http协议中，其实并没有对url长度作出限制
	url的最大长度和用户浏览器和Web服务器有关

一语惊醒梦中人啊

知道了原因问题就好解决了,由于我们是服务器间请求,不涉及浏览器等

所以只需要修改tomcat配置,使其支持更长的url就可以了

	<Connector URIEncoding="UTF-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" maxHttpHeaderSize="65536"/>

最后的maxHttpHeaderSize属性就是url最大长度啦

问题解决

---

## 附录

	Microsoft Internet Explorer (Browser)
	IE浏览器对URL的最大限制为2083个字符，如果超过这个数字，提交按钮没有任何反应

	Firefox (Browser)
	对于Firefox浏览器URL的长度限制为65,536个字符

	Safari (Browser)
	URL最大长度限制为 80,000个字符

	Opera (Browser)
	URL最大长度限制为190,000个字符

	Google (chrome)
	URL最大长度限制为8182个字符

	Apache (Server)
	能接受最大url长度为8,192个字符

	Microsoft Internet Information Server(IIS)
	能接受最大url的长度为16,384个字符

	ubuntu14.04 + tomcat 7.0.54
	url最大长度7509
