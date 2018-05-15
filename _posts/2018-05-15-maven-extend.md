---
layout: post
title:  Maven 继承
date:   2018-05-15 20:08:54
categories: Maven
---

* content
{:toc}

[参考链接](http://yanan0628.iteye.com/blog/2270411)

### 常用的继承元素

	groupId ：项目组 ID ，项目坐标的核心元素；    
	version ：项目版本，项目坐标的核心元素；    
	description ：项目的描述信息；    
	properties ：自定义的 Maven 属性；    
	dependencies ：项目的依赖配置；    
	dependencyManagement ：醒目的依赖管理配置；    
	repositories ：项目的仓库配置；    
	build ：包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等；
	
### 父 pom

	<project xmlns="http://maven.apache.org/POM/4.0.0"
	  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	  <modelVersion>4.0.0</modelVersion>
	  <groupId>com.sohu.train</groupId>
	  <artifactId>maven-aggregate</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
	  <packaging>pom</packaging>
	
	  <!-- 子模块 -->
	  <modules>
	    <module>../maven-01</module>
	    <module>../maven-02</module>
	    <module>../maven-03</module>
	  </modules>
	  <!-- 统一配置构件的版本号 -->
	  <properties>
	    <junit.version>3.8.1</junit.version>
	  </properties>
	
	  <!-- 依赖管理 -->
	  <dependencyManagement>
	    <dependencies>
	      <dependency>
	        <groupId>junit</groupId>
	        <artifactId>junit</artifactId>
	        <version>${junit.version}</version>
	        <scope>test</scope>
	      </dependency>
	    </dependencies>
	  </dependencyManagement>
	</project>

### 子 pom

	<project xmlns="http://maven.apache.org/POM/4.0.0"
	  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	  <modelVersion>4.0.0</modelVersion>
	  <!-- 指定父pom的坐标及pom位置 -->
	  <parent>
	    <groupId>com.sohu.train</groupId>
	    <artifactId>maven-aggregate</artifactId>
	    <version>0.0.1-SNAPSHOT</version>
	    <relativePath>../maven-aggregate/pom.xml</relativePath>
	  </parent>
	  <artifactId>maven-03</artifactId>
	  <packaging>jar</packaging>
	  <!-- 添加对junit依赖，这样公用配置只需要在maven-aggregate中去配置 -->
	  <dependencies>
	    <dependency>
	      <groupId>junit</groupId>
	      <artifactId>junit</artifactId>
	    </dependency>
	  </dependencies>
	</project> 

