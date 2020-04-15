---
layout: post
title:  MethodReferences
date:   2018-01-16 17:33:05
categories: JDK
---

* content
{:toc}

JDK1.8增加了一个方法引用（Method References）的新特性，现在就来探究一下其使用场景。  
原文来自[这里](https://www.cnblogs.com/JohnTsai/p/5806194.html)
	
### 形式

方法引用的标准形式就是：```类名::方法名```  
有如下四种形式的方法引用：

|类型|示例|
|----------|---|
|引用静态方法|ContainingClass::staticMethodName|
|引用某个对象的实例方法|containingObject::instanceMethodName|
|引用某个类型的任意对象的实例方法|ContainingType::methodName|
|引用构造方法|ClassName::new|

### 实例

定义两个类 Person 和 ComparisonProvider

	public class Person {
		Date age;
	
		public static int compareByAge(Person p1, Person p2) {
			return p1.age.compareTo(p2.age);
		}
	}
	
	public class ComparisonProvider {
	
		public int compare(Person p1, Person p2) {
			return p1.age.compareTo(p2.age);
		}
	}

1. 引用静态方法：

		Person[] persons = new Person[10];
		// 匿名内部类
		Arrays.sort(persons, new Comparator<Person>() {
			@Override
			public int compare(Person p1, Person p2) {
				return p1.age.compareTo(p2.age);
			}
		});
		// lambda表达式
		Arrays.sort(persons, (p1, p2) -> p1.age.compareTo(p2.age));
		// lambda和类的静态方法
		Arrays.sort(persons, (p1, p2) -> Person.compareByAge(p1, p2));
		// 方法引用，静态方法
		Arrays.sort(persons, Person::compareByAge);
	
2. 引用某个对象的实例方法：
	
		Person[] persons = new Person[10];
		ComparisonProvider provider = new ComparisonProvider();
		// lambda表达式
		Arrays.sort(persons, (p1, p2) -> p1.age.compareTo(p2.age));
		// 方法引用，对象的实例方法
		Arrays.sort(persons, provider::compare);
	
3. 引用某个类型的任意对象的实例方法：
	
		String[] strings = { "Hello", "World" };
		// 匿名内部类
		Arrays.sort(strings, new Comparator<String>() {
			@Override
			public int compare(String s1, String s2) {
				return s1.compareTo(s2);
			}
		});
		// lambda表达式
		Arrays.sort(strings, (s1, s2) -> s1.compareTo(s2));
		// 方法引用，类型的任意对象的实例方法
		Arrays.sort(strings, String::compareTo);
	
4. 引用构造方法

		引用的链接里有例子，但我觉得不好，我自己又写不出来，囧
	