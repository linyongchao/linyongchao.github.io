---
layout: post
title:  "tomcat 安全”
date:   2016-02-09 22:54:54
categories: tomcat
---

* content
{:toc}

## 8005端口

server.xml中有这样一句：

	<Server port="8005" shutdown="SHUTDOWN">

我们通过 telnet 命令访问 8005 端口并执行 SHUTDOWN（区分大小写）

	telnet 127.0.0.1 8005
	SHUTDOWN

不需要任何验证即可将 tomcat 关闭！

优化方式：

1. 更换端口
2. 更换关闭命令


## 错误页面

若使用 tomcat 默认错误页面，比如 404，则会暴露服务器版本等信息
为避免该情况，可以自定义错误页面：

1. web.xml 倒数第二行增加配置

	<error-page>
		<error-code>404</error-code>
		<location>/index.jsp</location>
	</error-page>

2. webapps/ROOT 下增加错误页面 index.jsp

## 删除自带信息

tomcat/webapps 下的所有文件夹