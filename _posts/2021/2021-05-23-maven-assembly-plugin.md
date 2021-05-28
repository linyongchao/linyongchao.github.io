---
layout: post
title:  maven-assembly-plugin 误修改文件
date:   2021-05-23 17:19:05
categories: Maven
---

* content
{:toc}

参考[这里](https://www.jianshu.com/p/5ec04f7b69b0)

### 问题

在项目中增加了一个 zip 文件，用 Maven 打包之后发现解压不了，用 MD5 校验后发现已经被修改了

### 解决

查询之后发现将 filtered 字段修改为 false 即可，如下：

	<fileSet>
	    <directory>src/main/resources</directory>
	    <filtered>false</filtered>
	    <includes>
	        <include>all.zip</include>
	    </includes>
	    <outputDirectory>/</outputDirectory>
	</fileSet>

### 原因

```maven-assembly-plugin```插件会对 filtered = true 的文件进行过滤，对其中的变量进行替换，这个过程会导致文件发现变化，禁止即可

### filtering 的作用

参考[这里](https://blog.csdn.net/lzhcoder/article/details/110807572)进行补充

Maven 的占位符解析表达式的使用场合默认只在 pom 文件范围内活动，如果想扩大它的活动范围，就必须指定需要扩大到哪些文件，然后指定 filtering = true  
Spring EL 表达式和 Maven 的占位符表达式长得一样，但两者默认井水不犯河水，不能在 Spring 的范围内取 Maven 的参数  
filtering 的作用就是打通两者的连接，使 Spring 的范围内能取到 Maven 的参数  
filtering 的使用要配合 resource，前者开启打通连接，后者指定打通的范围
