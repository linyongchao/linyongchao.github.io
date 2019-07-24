---
layout: post
title:  Spring 的单例模式和多例模式
date:   2019-07-19 22:06:54
categories: Java
---

* content
{:toc}

原文见[这里](https://blog.csdn.net/luanxiyuan/article/details/80560296)

Spring 的 Bean 的作用域可以由 scope 标签进行设置，共有四种：

	singleton	单例模式，默认值
	prototype	多例模式
	session	实例的作用域为 session 范围
	request	实例的作用域为 request 范围

Spring 的单例模式默认是饿汉式，即启动容器时为所有 Spring 配置文件中定义的 Bean 都生成一个实例  
其饿汉式和懒汉式的设置，可以通过如下两个```lazy-init```标签进行设置

	<beans default-lazy-init="true">
	<bean lazy-init="true">

Spring 的单例是相对于容器的，即在 ApplicationContext 中是单例的。而平常说的单例是相对于JVM的。  
一个 JVM 可以有多个 Spring 容器，而且 Spring 中的单例也只是按 Bean 的 id 来区分的。

* 用单例，是因为没必要为每个请求都新建一个对象，这样既浪费 CPU 又浪费内存
* 用多例，是为了防止并发问题；即一个请求改变了对象的状态，此时对象又处理另一个请求，而之前请求对对象状态的改变导致了对象对另一个请求做了错误的处理
* 用单例和用多例的标准：当对象含有可改变的状态时（更精确的说就是在实际应用中该状态会改变），则多例，否则单例；