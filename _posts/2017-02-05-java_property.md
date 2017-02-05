---
layout: post
title:  "JDK和eclipse配置"
date:   2017-02-05 17:16:54
categories: Java eclipse
---

* content
{:toc}

今天升级了下JDK和eclipse，发现好多配置都找不到位置，囧  
特记录如下：

## JDK环境变量

	cd ~
	vi .bash_profile

输入JDK环境变量配置内容：

	export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home
	export PATH=$JAVA_HOME/bin:$PATH
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

保存退出后，更新配置文件：

	source .bash_profile

## 折叠包

【Project Explorer】-> 左侧向下的小三角 ->【Package Presentation】->【Hierarchical】

## 单行代码的最大宽度

【Preferences】->【Java】->【Code Style】->【Formatter】，点击“new”新建一个“Profile”->【Line Wrapping】->【Maximum line width】

## 编译/运行环境设置

1. 【Preferences】->【Java】->【Compiler】
2. 【Preferences】->【Java】->【Installed JREs】

## 工作空间编码

【Preferences】->【General】->【Workspace】

## 文件编码

【Preferences】->【General】->【ContentTypes】

## jsp编码

【Preferences】->【Web】->【JSP Files】

## tab转为4个空格

1. 【Preferences】->【General】->【Editors】->【Text Editors】
2. 【Preferences】->【Java】->【Code Style】->【Formatter】新建profile,点击edit选择Tab policy为SpaceOnly

## 保存时自动引包、格式化

【Preferences】->【Java】->【Editor】->【Save Actions】

## 字体改变

【Preferences】->【General】->【Appearance】->【Colors and Fonts】->【Basic】->【Text Font】

## 不检查xml

【Preferences】->【Validation】中将XML开头的勾全部去除

## 不检查javascript

【Preferences】->【JavaScript】->【Validator】

## 常用插件

JBossTool(Hibernate插件)  Spring-sources-tools(Spring插件)  FindBugs