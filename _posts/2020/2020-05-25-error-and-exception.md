---
layout: post
title:  Error 和 Exception
date:   2020-05-25 18:59:54
categories: Java
---

* content
{:toc}

本文参考：[这里](https://blog.csdn.net/sjmz30071360/article/details/80016972)、[这里](https://blog.csdn.net/weixin_42124070/article/details/80833629)

### 关系

Error 和 Exception 都继承自 Throwable，在 Java 中只有 Throwable 类型的实例才可以被抛出（throw）或者捕获（catch），它是异常处理机制的基本组成类型。  
Error 是 Throw 不是 Exception。

![正常情况](https://linyongchao.github.io/static/img/error_and_exception.png)

### 区别

* Error：表示由 JVM 所侦测到的无法预期的错误，由于这是属于 JVM 层次的严重错误，导致 JVM 无法继续执行，因此是无法捕获的，无法采取任何恢复操作，顶多只能显示错误信息。比如：OutOfMemoryError、NoClassDefFoundError 等。
* Exception：表示可恢复的异常，是可以捕获的。

### Exception 分类

Java 异常主要分为两类：checked exception 和 RuntimeException  

* checked 异常就是常见的 IO 异常、SQL 异常等。对于这类异常，Java 编译器强制要求必须对这些异常进行处理。
* RuntimeException 也称为运行时异常，可以不处理。当出现这类异常时，会由虚拟机接管。比如 NullPointerException 异常，就是运行时异常。

### 抛出异常

抛出异常有三种形式：throw、throws 和系统自动抛异常

* 系统自动抛异常
	当程序语句出现一些逻辑错误、类型转换错误时，系统会自动抛异常，比如：
		public static void main(String[] args) {
		    int a = 5, b = 0;
		    System.out.println(a / b);
		}
	系统就会自动抛异常：
		Exception in thread "main" java.lang.ArithmeticException: / by zero
			at com.Test.main(Test.java:6)


* throw
	一般会用于程序出现某种逻辑时程序员主动抛出某种特定类型的异常，比如：
	    public static void main(String[] args) {
	        String str = "abc";
	        if (!StringUtils.isNumeric(str)) {
	            throw new NumberFormatException();
	        } else {
	            System.out.println(Integer.valueOf(str));
	        }
	    }
	字符串无法转成数字则会抛异常：
		Exception in thread "main" java.lang.NumberFormatException
			at com.Test.main(Test.java:9)

* throws
	是方法可能抛异常的声明，然后将异常交给上层调用他的程序处理，比如：
	    public static void main(String[] args) {
	        String str = "abc";
	        try {
	            System.out.println(transfer(str));
	        } catch (NumberFormatException e) {
	            System.out.println("数字转换失败");
	        }
	    }
	
	    public static int transfer(String str) throws NumberFormatException {
	        return Integer.parseInt(str);
	    }
	输出结果：```数字转换失败```

* throws 和 throw 的区别
	1. throws 出现在方法头；throw 出现在方法体
	2. throws 表示出现异常的可能性，并不一定会发生异常；throw 是抛异常，若执行了 throw 则一定会抛出某种异常
	3. 两者都是消极处理异常的方式（这里的消极并不是说这种方式不好），只是抛出或者可能抛出异常，但是不会由函数去处理异常，真正的处理异常由函数的上层调用处理

### 面试题一

有一个比较经典的面试题目，就是 NoClassDefFoundError 和 ClassNotFoundException 有什么区别？

1. NoClassDefFoundError 是 Error，ClassNotFoundException 是 Exception
2. NoClassDefFoundError 是 JVM 运行时通过 classpath 加载类时，找不到对应的类而抛出的错误。ClassNotFoundException 是在编译过程中如果可能出现此异常，在编译过程中必须将 ClassNotFoundException 异常抛出
3. NoClassDefFoundError 发生场景如下：
	* 类依赖的 class 或者 jar 不存在（简单说就是 maven 生成运行包后被篡改）
	* 类文件存在，但是存在不同的域中（简单说就是引入的类不在对应的包下)
	* 大小写问题，javac 编译的时候是无视大小的，有可能编译出来的 class 文件与想要的不一样
4. ClassNotFoundException 发生场景如下：
	* 调用 class 的 forName 方法时，找不到指定的类
	* ClassLoader 中的 findSystemClass() 方法时，找不到指定的类

### 面试题二

return 的值在 finally 中有修改，返回修改前还是修改后的值？

	public static void main(String[] args) {
		// 返回值为 4
	    System.out.println(get());

		// 返回值为 3
	    System.out.println(get2());
	}
	
	public static int get() {
	    try {
	        return 3;
	    } catch (Exception e) {
	        e.printStackTrace();
	    } finally {
	        return 4;
	    }
	}
	
	public static int get2() {
	    int x = 3;
	    try {
	        return x;
	    } catch (Exception e) {
	        e.printStackTrace();
	    } finally {
	        x++;
	    }
	    return 0;
	}
