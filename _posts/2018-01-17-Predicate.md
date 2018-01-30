---
layout: post
title:  Predicate
date:   2018-01-17 17:33:05
categories: JDK
---

* content
{:toc}

JDK1.8增加了一个Predicate接口，现在就来探究一下。

### 后记

除了支持一个泛型参数的 Predicate 接口  
还有接收对应类型的 DoublePredicate、IntPredicate、LongPredicate  
还有接收两个泛型参数的 BiPredicate 接口
	
### 源码

	/**
	 * Represents a predicate (boolean-valued function) of one argument.
	 *
	 * <p>This is a <a href="package-summary.html">functional interface</a>
	 * whose functional method is {@link #test(Object)}.
	 *
	 * @param <T> the type of the input to the predicate
	 *
	 * @since 1.8
	 */
	@FunctionalInterface
	public interface Predicate<T> {
	
	    /**
	     * Evaluates this predicate on the given argument.
	     *
	     * @param t the input argument
	     * @return {@code true} if the input argument matches the predicate,
	     * otherwise {@code false}
	     */
	    boolean test(T t);
	
	    /**
	     * Returns a composed predicate that represents a short-circuiting logical
	     * AND of this predicate and another.  When evaluating the composed
	     * predicate, if this predicate is {@code false}, then the {@code other}
	     * predicate is not evaluated.
	     *
	     * <p>Any exceptions thrown during evaluation of either predicate are relayed
	     * to the caller; if evaluation of this predicate throws an exception, the
	     * {@code other} predicate will not be evaluated.
	     *
	     * @param other a predicate that will be logically-ANDed with this
	     *              predicate
	     * @return a composed predicate that represents the short-circuiting logical
	     * AND of this predicate and the {@code other} predicate
	     * @throws NullPointerException if other is null
	     */
	    default Predicate<T> and(Predicate<? super T> other) {
	        Objects.requireNonNull(other);
	        return (t) -> test(t) && other.test(t);
	    }
	
	    /**
	     * Returns a predicate that represents the logical negation of this
	     * predicate.
	     *
	     * @return a predicate that represents the logical negation of this
	     * predicate
	     */
	    default Predicate<T> negate() {
	        return (t) -> !test(t);
	    }
	
	    /**
	     * Returns a composed predicate that represents a short-circuiting logical
	     * OR of this predicate and another.  When evaluating the composed
	     * predicate, if this predicate is {@code true}, then the {@code other}
	     * predicate is not evaluated.
	     *
	     * <p>Any exceptions thrown during evaluation of either predicate are relayed
	     * to the caller; if evaluation of this predicate throws an exception, the
	     * {@code other} predicate will not be evaluated.
	     *
	     * @param other a predicate that will be logically-ORed with this
	     *              predicate
	     * @return a composed predicate that represents the short-circuiting logical
	     * OR of this predicate and the {@code other} predicate
	     * @throws NullPointerException if other is null
	     */
	    default Predicate<T> or(Predicate<? super T> other) {
	        Objects.requireNonNull(other);
	        return (t) -> test(t) || other.test(t);
	    }
	
	    /**
	     * Returns a predicate that tests if two arguments are equal according
	     * to {@link Objects#equals(Object, Object)}.
	     *
	     * @param <T> the type of arguments to the predicate
	     * @param targetRef the object reference with which to compare for equality,
	     *               which may be {@code null}
	     * @return a predicate that tests if two arguments are equal according
	     * to {@link Objects#equals(Object, Object)}
	     */
	    static <T> Predicate<T> isEqual(Object targetRef) {
	        return (null == targetRef)
	                ? Objects::isNull
	                : object -> targetRef.equals(object);
	    }
	}
	
### 翻译
	
	/**
	 * 表示对输入参数进行断言（布尔值函数）
	 * 
	 * 这是一个函数式接口，其函数式方法为test
	 *
	 * @param <T> 断言的输入类型
	 *
	 * @since 1.8
	 */
	@FunctionalInterface
	public interface Predicate<T> {
	
	    /**
	     * 对给定参数进行断言求值
	     *
	     * @param t 输入参数
	     * @return 如果输入参数符合断言，则返回true，否则返回false
	     */
	    boolean test(T t);
	
	    /**
	     * 返回一个由当前断言和other断言组成的短路逻辑与断言组合
	     * 当对断言组合求值时，如果当前断言返回false，则不执行other断言
	     *
	     * 任意断言执行过程中抛出的异常将被反馈到断言调用者
	     * 如果当前断言执行过程中抛出异常，这other断言不会被执行
	     *
	     * @param other 一个要和当前断言组合成逻辑与断言的断言
	     * @return 一个由当前断言和other断言组成的短路逻辑与断言组合
	     * @throws NullPointerException 如果other断言为null则抛出空指针异常
	     */
	    default Predicate<T> and(Predicate<? super T> other) {
	        Objects.requireNonNull(other);
	        return (t) -> test(t) && other.test(t);
	    }
	
	    /**
	     * 返回一个和当前断言逻辑相反的断言
	     *
	     * @return 一个和当前断言逻辑相反的断言
	     */
	    default Predicate<T> negate() {
	        return (t) -> !test(t);
	    }
	
	    /**
	     * 返回一个由当前断言和other断言组成的短路逻辑或断言组合
	     * 当对断言组合求值时，如果当前断言返回true，则不执行other断言
	     *
	     * 任意断言执行过程中抛出的异常将被反馈到断言调用者
	     * 如果当前断言执行过程中抛出异常，这other断言不会被执行
	     *
	     * @param other 一个要和当前断言组合成逻辑或断言的断言
	     * @return 一个由当前断言和other断言组成的短路逻辑或断言组合
	     * @throws NullPointerException 如果other断言为null则抛出空指针异常
	     */
	    default Predicate<T> or(Predicate<? super T> other) {
	        Objects.requireNonNull(other);
	        return (t) -> test(t) || other.test(t);
	    }
	
	    /**
	     * 返回一个根据Objects#equals方法判断两个参数是否equal的断言
	     *
	     * @param <T> 断言的参数类型
	     * @param targetRef 用于比较相等的对象引用，有可能为null
	     * @return 一个根据Objects#equals方法判断两个参数是否equal的断言
	     */
	    static <T> Predicate<T> isEqual(Object targetRef) {
	        return (null == targetRef)
	                ? Objects::isNull
	                : object -> targetRef.equals(object);
	    }
	}
	
### 摘要

1. Predicate接口是函数式接口，其test方法用于返回一个boolean值
2. Predicate接口的三个默认方法即与、或、非操作，静态方法为isEqual操作

### 实例

定义一个对象：

	public class Student {
	
		String name;
		Sex sex;
		int grade;
		// 基础学费
		double tuition = 10000.0;
	
		public Student(String name, Sex sex, int grade) {
			this.name = name;
			this.sex = sex;
			this.grade = grade;
		}
	
		enum Sex {
			MALE, FEMALE
		}
	
		public void actualTuition(Student student, Predicate<Student> isPass, Consumer<Student> discount) {
			// 一旦考试及格就可以学费打折
			if (isPass.test(student)) {
				discount.accept(student);
			}
		}
	
		@Override
		public String toString() {
			return "Student [name=" + name + ", sex=" + sex + ", grade=" + grade + ", tuition=" + tuition + "]";
		}
	
	}
	
实例1，test、negate、isEqual：
	
	Student s1 = new Student("lin", Sex.MALE, 80);
	Student s2 = new Student("chen", Sex.FEMALE, 50);

	Predicate<Student> grade = s -> s.grade > 60;

	System.out.println("test1=" + grade.test(s1));
	System.out.println("test2=" + grade.test(s2));

	System.out.println("negate1=" + grade.negate().test(s1));
	System.out.println("negate2=" + grade.negate().test(s2));

	Predicate<Student> equal = Predicate.isEqual(s1);
	System.out.println("isEqual1=" + equal.test(s1));
	System.out.println("isEqual2=" + equal.test(s2));
	
	输出结果为：
	test1=true
	test2=false
	negate1=false
	negate2=true
	isEqual1=true
	isEqual2=false
	
实例2，and、or：
	
	Student s1 = new Student("lin", Sex.MALE, 80);
	Student s2 = new Student("chen", Sex.FEMALE, 50);

	Predicate<Student> grade = s -> s.grade > 60;
	Predicate<Student> sex = s -> s.sex == Sex.FEMALE;
	Predicate<Student> name = s -> s.name.startsWith("c");

	Predicate<Student> and = grade.and(sex).and(name);
	Predicate<Student> or = grade.or(sex).or(name);

	System.out.println("and1=" + and.test(s1));
	System.out.println("and2=" + and.test(s2));

	System.out.println("or1=" + or.test(s1));
	System.out.println("or2=" + or.test(s2));
	
	输出结果为：
	and1=false
	and2=false
	or1=true
	or2=true

实例3，Predicate 和 Consumer：

	Student s1 = new Student("lin", Sex.MALE, 80);
	Student s2 = new Student("chen", Sex.FEMALE, 50);
	System.out.println("计算前：" + s1.toString());
	System.out.println("计算前：" + s2.toString());

	Predicate<Student> isPass = s -> s.grade >= 60;
	Consumer<Student> discount = s -> s.tuition = s.tuition * (100 - s.grade) / 100;

	s1.actualTuition(s1, isPass, discount);
	s2.actualTuition(s2, isPass, discount);
	System.out.println("计算后：" + s1.toString());
	System.out.println("计算后：" + s2.toString());
	
	输出结果为：
	计算前：Student [name=lin, sex=MALE, grade=80, tuition=10000.0]
	计算前：Student [name=chen, sex=FEMALE, grade=50, tuition=10000.0]
	计算后：Student [name=lin, sex=MALE, grade=80, tuition=2000.0]
	计算后：Student [name=chen, sex=FEMALE, grade=50, tuition=10000.0]
