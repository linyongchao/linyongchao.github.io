---
layout: post
title:  WebService
date:   2018-08-30 15:46:54
categories: Java
---

* content
{:toc}

参考[这里](https://blog.csdn.net/qazwsxpcm/article/details/70370490)和[这里](http://iplaybest.com/webservice_one.html)  
代码见[这里](https://github.com/linyongchao/WebServiceTest)

## Service
1. ```@WebService```注解指定服务
2. ```Endpoint.publish()```方法发布服务
3. ```@WebMethod(exclude= true)```注解指定的方法不会发布
4. 静态方法不会发布

### 服务类
	
	/**
	 * JDK1.6之后可利用@WebService注解发布WebService
	 */
	@WebService
	public class MathService {
		/**
		 * hello
		 */
		public String hello(String name) {
			return "hello " + name + "!";
		}
	
		/**
		 * 加法
		 */
		public int add(int a, int b) {
			return a + b;
		}
	
		/**
		 * 减法，因为@WebMethod(exclude= true)注解，所以不会被发布
		 */
		@WebMethod(exclude = true)
		public int subtract(int a, int b) {
			return a - b;
		}
	
		/**
		 * 乘法，因为是静态的，所以不会发布
		 */
		public static int multiply(int a, int b) {
			return a * b;
		}
	}

### 发布类

	public class Publish {
		public static void main(String[] args) {
			Endpoint.publish("http://localhost:8088/MathService", new MathService());
			System.out.println("服务发布成功");
		}
	}

## Client

1. Eclipse可以自动生成Client代码
2. Eclipse生成的代码通过```new ……ServiceLocator().get……Port()```获取调用类
3. wsimport命令可以自动生成Client代码
4. wsimport命令生成的代码通过```new ……Service().get……Port()```获取调用类

### Eclipse

在项目上右键-->New-->Other...-->Web Services-->Web Service Client-->Next-->Service definition填入服务路径后点击Finish即可

### wsimport
格式：

	wsimport -s [结果路径] -p [包名] -keep [服务路径]  
	
示例：

	wsimport -s /Users/lin/Documents/git/WebServiceTest/Client/src -p my.learn.wsimport -keep http://localhost:8088/MathService\?wsdl