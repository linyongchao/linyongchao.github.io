---
layout: post
title:  大补课【2】switch
date:   2020-01-09 23:45:54
categories: Java
---

* content
{:toc}

### 前言

今天写一个 switch 判断，类型是枚举，结果报错，就补了下课  
参考[这里](https://blog.csdn.net/qq_37893505/article/details/91538833)和[这里](https://blog.csdn.net/zwt0909/article/details/52701283)

### 支持类型

|JDK版本|支持类型|变化|
|:---:|:---:|:---:|
|<1.5|byte、short、int、char|--|
|1.5|byte、short、int、char、enum|增加枚举|
|1.7|byte、short、int、char、enum、String|增加字符串|

* switch 支持的类型是逐渐增加的
* 也支持四种基本类型的封装类
* 封装类不能为 null，否则会报空指针异常

### 实现机制

* switch 底层是使用 int 类型来判断的
* int 类型是四个字节的整数型类型，只要字节小于或等于 4 的整数型类型都是可以转化成 int 类型，所以支持 byte（1字节），short（2字节）
* long（8字节）超出了 int 的范围，因而不支持
* 枚举和字符(串)也是转化为 int 类型间接实现的

### 错误解决

今天写的 switch 报错如下：  
	
	An enum switch case label must be the unqualified name of an enumeration constant

即：case 后的枚举类型必须使用非限定名，那么去掉就解决问题了

