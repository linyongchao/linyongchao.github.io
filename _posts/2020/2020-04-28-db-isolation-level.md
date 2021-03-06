---
layout: post
title:  事务的隔离级别
date:   2020-04-28 18:59:54
categories: DB
---

* content
{:toc}

本文参考：[这里](https://www.cnblogs.com/wyaokai/p/10921323.html)、[这里](http://blog.itpub.net/26736162/viewspace-2638951/)

### 事务的并发问题

1. 脏读：事务 A 读取了事务 B 已修改但尚未提交的数据。当事务 B 正在多次修改某个数据且未提交时，事务 A 来访问数据，就会造成两个事务得到的数据不一致，此时事务 A 读取到的数据就是脏数据。
2. 不可重复读：在同一个事务中，同一个事务在 A 时间读取某一行，在 B 时间重新读取这一行时，发现这一行已经发生了修改，被更新了或被删除了。
3. 幻读（也叫虚读）：在同一个事务中，当同一查询多次执行时，由于其他插入操作的事务提交，导致每次返回不同的结果集。幻读是事务非独立执行时发生的一种现象。

小结：

1. 脏读和不可重复读的区别：脏读是一个事务读取了另一个事务未提交的脏数据；不可重复读是在同一个事务范围内多次查询同一条记录却返回了不同的数据值，这是由于在查询间隔期间，该数据被另一个事务修改并提交了。
2. 幻读和不可重复读的区别：幻读和不可重复读都是读取了另一个事务中已经提交的数据。不同的是：不可重复读多次查询的都是同一条记录，针对的是对同一条记录的修改或删除；幻读是针对一个数据整体，比如数据的条数，主要是插入操作。
3. 不可重复读的解决方案：不可重复读是事务并发修改同一条记录导致的，所以可以考虑加行锁，这会导致锁冲突加剧，影响性能；也可以考虑通过 MVCC 在无锁的情况下避免不可重复读。
4. 幻读的解决方案：幻读是事务并发增加导致的，无法通过加锁解决问题，因为对新增的记录根本无法加锁。需要将事务串行化，才能避免幻读。

### 事务隔离级别

|事务隔离级别|脏读|不可重复读|幻读|
|:-:|:-:|:-:|:-:|
|读未提交（read-uncommitted）|是|是|是|
|读已提交（read-committed）|否|是|是|
|可重复读（repeatable-read）|否|否|是|
|串行化（serializable）|否|否|否|

小结：不同的隔离级别有不同的现象，并有不同的锁和并发机制，隔离级别越高，数据库的并发性能就越差。



