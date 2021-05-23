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
