---
layout: post
title:  FunctionalInterface
date:   2018-01-10 17:33:05
categories: JDK
---

* content
{:toc}

JDK1.8增加了一个函数式接口特性，并且增加了一个函数式接口注解，现在就来探究一下什么是函数式接口。
	
### 源码

	/**
	 * An informative annotation type used to indicate that an interface
	 * type declaration is intended to be a <i>functional interface</i> as
	 * defined by the Java Language Specification.
	 *
	 * Conceptually, a functional interface has exactly one abstract
	 * method.  Since {@linkplain java.lang.reflect.Method#isDefault()
	 * default methods} have an implementation, they are not abstract.  If
	 * an interface declares an abstract method overriding one of the
	 * public methods of {@code java.lang.Object}, that also does
	 * <em>not</em> count toward the interface's abstract method count
	 * since any implementation of the interface will have an
	 * implementation from {@code java.lang.Object} or elsewhere.
	 *
	 * <p>Note that instances of functional interfaces can be created with
	 * lambda expressions, method references, or constructor references.
	 *
	 * <p>If a type is annotated with this annotation type, compilers are
	 * required to generate an error message unless:
	 *
	 * <ul>
	 * <li> The type is an interface type and not an annotation type, enum, or class.
	 * <li> The annotated type satisfies the requirements of a functional interface.
	 * </ul>
	 *
	 * <p>However, the compiler will treat any interface meeting the
	 * definition of a functional interface as a functional interface
	 * regardless of whether or not a {@code FunctionalInterface}
	 * annotation is present on the interface declaration.
	 *
	 * @jls 4.3.2. The Class Object
	 * @jls 9.8 Functional Interfaces
	 * @jls 9.4.3 Interface Method Body
	 * @since 1.8
	 */
	@Documented
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.TYPE)
	public @interface FunctionalInterface {}
	
### 翻译
	
	一种含义丰富的注解类型，用于表明一个接口的类型有意被定义为Java语言规范定义的“函数式接口”。
	
	从概念上来说，函数式接口有且仅有一个抽象方法。
	如果一个方法拥有一个实现，那他不是抽象的。
	如果一个接口声明的抽象方法是重写的Object对象的方法，那么该抽象方法不列入抽象方法的统计中。因为该接口的任何实现都将从Object或者其他获取到这个实现。
	
	注意：函数式接口的实现可以由lambda表达式、方法引用、构造函数引用来创建。
	
	如果一个类型被定义为函数式接口，那么编译器将报错，除非：
	1. 被注解的类型是一个接口，而不是注解、枚举或类
	2. 该接口满足函数式接口的要求

	然而，编译器会将任何满足函数式接口定义的接口视为函数式接口，而不管这个接口在声明时是否添加了FunctionalInterface注解。
	
### 摘要

1. 该注解只能标记在“有且仅有一个抽象方法”的接口上
2. 只要接口符合函数式接口的定义，无论其是否拥有该注解，编译器都会将其视作函数式接口
3. 接口默认继承Object对象，如果一个接口的抽象方法是重写的Object对象中的方法，那么该抽象方法不计入抽象方法统计

上述1和2很好理解，关于3，我们举例说明

### 实例

实例1：

	/**
	 * 有且仅有一个抽象方法的接口可以声明为@FunctionalInterface
	 * 
	 * @author lin
	 * @date 2018年1月11日 下午6:48:10
	 */
	@FunctionalInterface
	public interface FunctionTest1 {
		public void test();
	}
	
实例2：
	
	/**
	 * 有多个抽象方法的接口无法被声明为@FunctionalInterface，会报错
	 * 
	 * @author lin
	 * @date 2018年1月11日 下午6:48:44
	 */
	// @FunctionalInterface
	public interface FunctionTest2 {
		public void test1();
	
		public void test2();
	}
	
实例3：
	
	/**
	 * 如果抽象方法是重写Object对象中的方法则不列入抽象方法的统计<br>
	 * FunctionTest3依然是一个有且仅有一个抽象方法的接口
	 * 
	 * @author lin
	 * @date 2018年1月11日 下午6:48:10
	 */
	@FunctionalInterface
	public interface FunctionTest3 {
		public void test();
	
		@Override
		int hashCode();
	
		@Override
		boolean equals(Object obj);
	
		String toString();
	}
	
实例4

	/**
	 * 静态方法和默认方法也不会认为是抽象方法
	 * 
	 * @author lin
	 * @date 2018年1月11日 下午6:48:10
	 */
	@FunctionalInterface
	public interface FunctionTest4 {
		public void test();
	
		default String getName() {
			return this.getName();
		}
	
		static void print() {
			System.out.println("now in FunctionTest4");
		}
	}