---
layout: post
title:  单例模式
date:   2019-12-25 19:15:54
categories: Java
---

* content
{:toc}

设计模式之前已经总结过了，只是形式是代码，现在重新理一下，整理为博客  
遗憾的是枚举实现单例其实并没有弄明白，网上也没有找到好点的例子，需要继续学习

### 第一种

	/**
	 * @Description: 懒加载，线程不安全
	 */
	public class SingletonOne {
		private static SingletonOne instance = null;
		private SingletonOne() {
		}
		public static SingletonOne getInstance() {
			if (instance == null) {
				instance = new SingletonOne();
			}
			return instance;
		}
	}

### 第二种

	/**
	 * @Description: 懒加载，线程安全，但性能较低
	 */
	public class SingletonTwo {
		private static SingletonTwo instance = null;
		private SingletonTwo() {
		}
		public static synchronized SingletonTwo getInstance() {
			if (instance == null) {
				instance = new SingletonTwo();
			}
			return instance;
		}
	}

### 第三种

	/**
	 * @Description: 懒加载，线程安全，且高性能，双重校验锁（DCL，即 double-checked locking）
	 */
	public class SingletonThree {
		private volatile static SingletonThree instance = null;
		private SingletonThree() {
		}
		public static SingletonThree getInstance() {
			if (instance == null) {
				synchronized (SingletonThree.class) {
					if (instance == null) {
						instance = new SingletonThree();
					}
				}
			}
			return instance;
		}
	}

### 第四种

	/**
	 * @Description: 懒加载，线程安全，使用静态内部类维护单例的实现，JVM内部的机制能够保证当一个类被加载的时候，这个类的加载过程是线程互斥的
	 */
	public class SingletonFour {
		private SingletonFour() {
		}
		/**
		 * 此处使用一个内部类来维护单例
		 **/
		private static class SingletonFactory {
			private static final SingletonFour instance = new SingletonFour();
		}
		public static SingletonFour getInstance() {
			return SingletonFactory.instance;
		}
	}

### 第五种

	/**
	 * @Description: 非懒加载，线程安全，JVM内部的机制能够保证当一个类被加载的时候，这个类的加载过程是线程互斥的
	 */
	public class SingletonFive {
		private static SingletonFive instance = new SingletonFive();
		private SingletonFive() {
		}
		public static SingletonFive getInstance() {
			return instance;
		}
	}

### 第六种

	/**
	 * @Description: 非懒加载，线程安全，实现单例模式的最佳方法，更简洁，自动支持序列化机制，绝对防止多次实例化
	 **/
	public enum SingletonSix {
	    INSTANCE;
	    public SingletonSix getInstance() {
	        return INSTANCE;
	    }
	}

### 结论

|方法|名称|懒加载|线程安全|性能|
|:-:|:-:|:-:|:-:|:-:|
|第一种|--|√|×|√|
|第二种|--|√|√|×|
|第三种|双重校验锁|√|√|√|
|第四种|静态内部类|√|√|√|
|第五种|饿汉式|×|√|√|
|第六种|枚举|×|√|√|

* 不能使用第一种（线程不安全）、第二种（性能较低）
* 建议使用第五种（饿汉式）、第六种（枚举）
* 明确需要懒加载使用第三种（双重校验锁）、第四种（静态内部类）

### 其他

一篇比较好的文章，推荐[这个](https://www.cnblogs.com/limingluzhu/p/5156659.html)  
枚举单例的优势见[这里](https://www.cnblogs.com/chiclee/p/9097772.html)