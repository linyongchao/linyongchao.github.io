---
layout: post
title:  Java 内存布局
date:   2020-12-29 16:59:54
categories: Java
---

* content
{:toc}

本文参考[这里](https://www.jianshu.com/p/d42ac3ab41f7)、[这里](https://zhuanlan.zhihu.com/p/50984945)

## 前言

问：```Object o = new Object();```占用多少字节？

## 数据类型

Java 中数据类型主要分为原生类型、类、接口、数组这几种。  
原生类型包括int、long、double、char、byte等。  
原生类型对应的包装类型如Integer、Long、Double这些在 Java 里面归类到类(class)里面，这里为了描述方便把接口(interface)也归类到类(class)里面。  
所以：Java 的数据类型主要归为三类：原生类型、类(class)、数组。  
其中类和数组在 Java 虚拟机里面是对象的形式的创建的，所以我们说的对象通常就是指类和数组的实例。

## 对象的组成

根据 Java 虚拟机规范里面的描述：Java 对象分为三部分：对象头(Object Header), 实例数据(instance data)，对齐填充(padding)。如图：

![CAP](https://linyongchao.github.io/static/img/memory-layout.jpg)

数组与对象类似，只是对象头部分多了代表存储长度的Length部分，其长度为4字节。

### 对象头（Object Header）

对象头分为两部分：Mark Word 与 Class Pointer(类型指针)  

* Mark Word 存储了对象的 hashCode、GC信息、锁信息三部分  
在32位 JVM 上占用4字节，64位 JVM 则占用8字节  
* Class Pointer 存储了指向类对象信息的指针  
在32位 JVM 上占用4字节，64位 JVM 则占用8字节  
* Length 只在数组对象中存在，用来记录数组的长度，占用4字节

另外，在64位 JVM 上有一个压缩指针选项```-XX:+UseCompressedOops```，默认是开启的。开启之后Class Pointer部分就会压缩为4字节，此时对象头总大小就会缩小到12字节。

### Instance data

对象实际数据，对象实际数据包括了对象的所有成员变量，其大小由各个成员变量的大小决定。  
这里不包括静态成员变量，因为其是在方法区维护的

### Padding

Java 对象占用空间是8字节对齐的，即所有 Java 对象占用字节数必须是8的倍数，padding 的作用就是补充字节，保证对象是8字节的整数倍。

## 结语

前言的问题答案：

1. 如果是32位 JVM，则 Mark Word 和 Class Pointer 各占用4字节，共计8字节
2. 如果是64位 JVM，没有开启指针压缩， Mark Word 和 Class Pointer 各占用8字节，共计16字节
3. 如果是64位 JVM，且开启指针压缩， Mark Word 占用8字节，Class Pointer 占用4字节，由于不是8的倍数，需要由 Padding 补充4字节，共计16字节
