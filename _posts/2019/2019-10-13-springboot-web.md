---
layout: post
title:  SpringBoot WEB
date:   2019-10-13 16:36:54
categories: Java
---

* content
{:toc}

我有一个将近两万张图片的文件夹，准备写个程序，用浏览器浏览图片，所以需要搭个 SpringBoot 的 WEB 架子  
参考[这里](https://www.jianshu.com/p/198497e08e00)

### pom.xml

	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-thymeleaf</artifactId>
	</dependency>

### application.yml

	spring:
	  thymeleaf:
	    prefix: classpath:/templates/
	    suffix: .html
	    mode: HTML5
	    encoding: UTF-8

### hello.html

在 ```resources``` 目录下创建目录 ```templates```，再创建文件 ```hello.html```，内容如下

	<!DOCTYPE html>
	<html lang="en">
	<head lang="en">
	    <meta charset="UTF-8"/>
	    <title>图片配置</title>
	</head>
	<body>
	<form action="/picture/init">
	    输入源：<input name="path" value=""/>
	    <input type="submit" value="提交"/>
	</form>
	</body>
	</html>

### PictureController.java

注意是 ```@Controller``` 注解，而不是 ```@RestController``` 注解

	@Controller
	@RequestMapping("/picture")
	public class PictureController {
	
	    /**
	     * 跳转配置页面
	     *
	     * @param
	     * @return
	     * @author lin
	     * @date 2019/10/13 16:04
	     **/
	    @RequestMapping("/hello")
	    public String hello() {
	        return "hello";
	    }
	
	}

启动后 ```http://localhost:12345/picture/hello``` 链接即可打开页面，架子就搭好了