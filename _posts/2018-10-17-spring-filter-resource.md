---
layout: post
title:  Spring Filter @Resource 无效
date:   2018-10-17 15:06:54
categories: Java
---

* content
{:toc}

原文见[这里](https://blog.csdn.net/YISHENGYOUNI95/article/details/80144842)

### 原因

Web 应用启动顺序为：Listener -> Filter -> Servlet  
即，项目启动时，先初始化 Listener，此时配置在 applicationContext.xml 里的 bean 会被初始化和注入；然后初始化 Filter；最后初始化 dispathServlet  
因此，在 Filter 内注入一个注解的 bean 时，会注入失败，因为 Filter 初始化时，注解的 bean 还没初始化

### 解决方案

手动注入，分为两步：

1. 首先在 application.xml 中声明 bean

		<bean id="mosLog" class="com.uusafe.mos.log.client.MosLog"></bean>

2. 然后在 LoginFilter 中获取 bean

		private IMosLog mosLog;
		ApplicationContext ac = WebApplicationContextUtils.getWebApplicationContext(request.getServletContext());
		mosLog = (IMosLog) ac.getBean("mosLog");