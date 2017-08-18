---
layout: post
title:  Ubuntu 设置 Tomcat 自动启动
date:   2016-02-28 19:57:05
categories: Ubuntu Tomcat
---

* content
{:toc}

## 环境

* Ubuntu 14.04
* tomcat-7.0.67

## 配置

在 /etc/rc.local 中增加两行：

	export JAVA_HOME=/usr/java/jdk1.7.0_79
	/usr/local/tomcat/bin/startup.sh
	exit 0


## 乱码

重启系统后，tomcat可以自动启动，但是中文乱码

修改 tomcat/bin/catalina.sh 234行：

	JAVA_OPTS="$JAVA_OPTS -server -Dfile.encoding=UTF-8"




