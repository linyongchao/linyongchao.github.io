---
layout: post
title:  Java 执行 class 文件时指定依赖包
date:   2025-02-09 07:49:05
categories: Java
---

* content
{:toc}

照样是对通义千问的答案进行整理后的结果  
话说通义千问给出的答案真啰嗦啊，明明两行命令的事情，给出的答案有两三屏……

### 问题

	/path/target/
	├── classes/com/B.class
	└── lib/A.jar

目录结构如上，B.class 依赖 A.jar ，那么如何运行 B.class ？

### 答案

	cd /path/target/
	java -cp classes:lib/A.jar com.B
	
### 备注

	-cp 是 -classpath 的缩写