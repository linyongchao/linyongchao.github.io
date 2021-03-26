---
layout: post
title:  修改IDEA内存配置
date:   2021-03-26 18:39:05
categories: IDEA
---

* content
{:toc}

本文参考[这里](https://baijiahao.baidu.com/s?id=1652945219104366994&wfr=spider&for=pc)  

## 位置

位于目录```/Applications/IntelliJ IDEA.app/Contents/bin```下的文件```idea.vmoptions```

## 原文

	-Xms128m
	-Xmx750m
	-XX:ReservedCodeCacheSize=240m
	-XX:+UseCompressedOops
	-Dfile.encoding=UTF-8
	-XX:+UseConcMarkSweepGC
	-XX:SoftRefLRUPolicyMSPerMB=50
	-ea
	-Dsun.io.useCanonCaches=false
	-Djava.net.preferIPv4Stack=true
	-Dfile.encoding=UTF-8
	-XX:+HeapDumpOnOutOfMemoryError
	-XX:-OmitStackTraceInFastThrow
	-Xverify:none
	
	-XX:ErrorFile=$USER_HOME/java_error_in_idea_%p.log
	-XX:HeapDumpPath=$USER_HOME/java_error_in_idea.hprof
	-Xbootclasspath/a:../lib/boot.jar
	
## 修改后

只改了两个：  
```-Xms Java``` Heap初始值  
```-Xmx Java``` Heap最大值  

	-Xms1280m
	-Xmx4096m
	-XX:ReservedCodeCacheSize=240m
	-XX:+UseCompressedOops
	-Dfile.encoding=UTF-8
	-XX:+UseConcMarkSweepGC
	-XX:SoftRefLRUPolicyMSPerMB=50
	-ea
	-Dsun.io.useCanonCaches=false
	-Djava.net.preferIPv4Stack=true
	-Dfile.encoding=UTF-8
	-XX:+HeapDumpOnOutOfMemoryError
	-XX:-OmitStackTraceInFastThrow
	-Xverify:none
	
	-XX:ErrorFile=$USER_HOME/java_error_in_idea_%p.log
	-XX:HeapDumpPath=$USER_HOME/java_error_in_idea.hprof
	-Xbootclasspath/a:../lib/boot.jar
