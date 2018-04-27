---
layout: post
title:  Spring Async 注解
date:   2018-04-25 19:55:54
categories: Java
---

* content
{:toc}

### 未配置不生效

只添加注解 @Async 不生效，需要在配置文件中加入异步支持：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	  xmlns:context="http://www.springframework.org/schema/context"
	  <!-- 下面的 schema/task 必须引入 -->
	  xmlns:task="http://www.springframework.org/schema/task"
	  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
	    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd
	    <!-- 下面的 schema/task 必须引入 -->
	    http://www.springframework.org/schema/task
	    http://www.springframework.org/schema/task/spring-task-3.1.xsd">
	
	  <context:component-scan base-package="com.xxx.*"/>
	  <!-- 支持异步方法执行 -->
	  <task:annotation-driven/>
	</beans>
	
### 覆盖不生效

上一步已经配置但注解 @Async 依然不生效，就考虑 XML 覆盖的问题  
比如：  
项目先加载 ```applicationContext.xml``` 文件（A）  
后加载 ```mvc-dispatcher-servlet.xml``` 文件（B）  
只在 A 文件中配置上一步内容就会被没有配置的 B 文件覆盖

解决办法如下：  
方法一：配置 B 文件的扫描范围，不扫描带有注解 @Async 的文件  
方法二：将 A 文件中的配置挪到 B 文件中

### 报错

报错如下：

	Only one AsyncAnnotationBeanPostProcessor may exist within the context.
	
原因是多个 XML 文件中都有 ```<task:annotation-driven/>```，但只允许有一个  
根据情况删除多余的即可