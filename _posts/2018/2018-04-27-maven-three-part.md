---
layout: post
title:  Maven 引入第三方包
date:   2018-04-27 16:00:54
categories: Maven
---

* content
{:toc}

## 确定包版本

以 ojdbc6.jar 为例  

1. 执行命令 ```tar xf ojdbc6.jar``` 解压 jar 包  
2. 读取 ```META-INF/MANIFEST.MF``` 文件，确定包版本为 11.1.0.7.0

		Manifest-Version: 1.0
		Implementation-Vendor: Oracle Corporation
		Implementation-Title: ojdbc6.jar
		Implementation-Version: Oracle JDBC Driver version - "11.1.0.7.0-Produ
		 ction"
		Implementation-Time: Thu Aug 28 17:39:02 2008
		Specification-Vendor: Oracle Corporation
		Sealed: true
		Created-By: 1.6.0 (Sun Microsystems Inc.)
		Specification-Title: Oracle JDBC driver classes for use with JDK6
		Specification-Version: Oracle JDBC Driver version - "11.1.0.7.0-Produc
		 tion"
		Main-Class: oracle.jdbc.OracleDriver
		
		Name: oracle/sql/converter_xcharset/
		Sealed: false
		
		Name: oracle/sql/
		Sealed: false
		
		Name: oracle/sql/converter/
		Sealed: false
	
## 本地库

执行如下命令将 jar 包添加到本地 Maven 库：

	mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.1.0.7.0 -Dpackaging=jar -Dfile=ojdbc6.jar

## 私库

执行如下步骤将 jar 包添加到 Nexus 私库：

1. 上传文件到私库：
![upload](https://linyongchao.github.io/static/img/maven-upload.png)
2. 获取引用
![get](https://linyongchao.github.io/static/img/maven-get-quote.png)
