---
layout: post
title:  redo log
date:   2021-09-21 12:19:05
categories: MySQL
---

* content
{:toc}

原文见[这里](https://mp.weixin.qq.com/s/z1qNuDBcQUHyXvWUxMvPFQ)

### 前言

说到```MySQL```，有两块日志一定绕不开，一个是```InnoDB```存储引擎的```redo log```（重做日志），另一个是```MySQL Servce```层的 ```binlog```（归档日志）。

![img](https://linyongchao.github.io/static/img/mysql-log/1.webp)

只要是数据更新操作，就一定会涉及它们，今天就来聊聊```redo log```（重做日志）。

### redo log

```redo log```（重做日志）是```InnoDB```存储引擎独有的，它让```MySQL```拥有了崩溃恢复能力。

比如```MySQL```实例挂了或宕机了，重启时，```InnoDB```存储引擎会使用```redo log```恢复数据，保证数据的持久性与完整性。

![img](https://linyongchao.github.io/static/img/mysql-log/2.webp)

```MySQL```中数据是以页为单位，查询一条记录，会从硬盘把一页的数据加载出来，加载出来的数据叫数据页，会放入到```Buffer Pool```中。

后续的查询都是先从```Buffer Pool```中找，没有命中再去硬盘加载，减少硬盘```IO```开销，提升性能。

更新表数据的时候，也是如此，发现```Buffer Pool```里存在要更新的数据，就直接在```Buffer Pool```里更新。

然后会把“在某个数据页上做了什么修改”记录到重做日志缓存（```redo log buffer```）里，接着刷盘到```redo log```文件里。

![img](https://linyongchao.github.io/static/img/mysql-log/3.webp)

理想情况，事务一提交就会进行刷盘操作，但实际上，刷盘的时机是根据策略来进行的。

ps：每条redo记录由“表空间号+数据页号+偏移量+修改数据长度+具体修改的数据”组成

### 刷盘时机

```InnoDB```存储引擎为```redo log```的刷盘策略提供了```innodb_flush_log_at_trx_commit```参数，它支持三种策略

* 设置为0的时候，表示每次事务提交时不进行刷盘操作
* 设置为1的时候，表示每次事务提交时都将进行刷盘操作（默认值）
* 设置为2的时候，表示每次事务提交时都只把redo log buffer内容写入page cache

另外```InnoDB```存储引擎有一个后台线程，每隔1秒，就会把```redo log buffer```中的内容写到文件系统缓存（```page cache```），然后调用```fsync```刷盘。

![img](https://linyongchao.github.io/static/img/mysql-log/4.webp)

也就是说，一个没有提交事务的```redo log```记录，也可能会刷盘。

为什么呢？

因为在事务执行过程```redo log```记录是会写入```redo log buffer```中，这些```redo log```记录会被后台线程刷盘。

![img](https://linyongchao.github.io/static/img/mysql-log/5.webp)

除了后台线程每秒1次的轮询操作，还有一种情况，当```redo log buffer```占用的空间即将达到```innodb_log_buffer_size```一半的时候，后台线程会主动刷盘。

下面是不同刷盘策略的流程图

1. ```innodb_flush_log_at_trx_commit=0```

	![img](https://linyongchao.github.io/static/img/mysql-log/6.webp)
	
	为0时，如果```MySQL```挂了或宕机可能会有1秒数据的丢失。
	
2. ```innodb_flush_log_at_trx_commit=1```

	![img](https://linyongchao.github.io/static/img/mysql-log/7.webp)
	
	为1时， 只要事务提交成功，```redo log```记录就一定在硬盘里，不会有任何数据丢失。
	
	如果事务执行期间```MySQL```挂了或宕机，这部分日志丢了，但是事务并没有提交，所以日志丢了也不会有损失。
	
3. ```innodb_flush_log_at_trx_commit=2```

	![img](https://linyongchao.github.io/static/img/mysql-log/8.webp)
	
	为2时， 只要事务提交成功，```redo log buffer```中的内容只写入文件系统缓存（```page cache```）。
	
	如果仅仅只是```MySQL```挂了不会有任何数据丢失，但是宕机可能会有1秒数据的丢失。
	
### 日志文件组

硬盘上存储的```redo log```日志文件不只一个，而是以一个日志文件组的形式出现的，每个的```redo```日志文件大小都是一样的。

比如可以配置为一组4个文件，每个文件的大小是1GB，整个```redo log```日志文件组可以记录4G的内容。

它采用的是环形数组形式，从头开始写，写到末尾又回到头循环写，如下图所示：

![img](https://linyongchao.github.io/static/img/mysql-log/9.webp)

这个日志文件组中还有两个重要的属性，分别是```write pos```、```checkpoint```

* ```write pos```是当前记录的位置，一边写一边后移
* ```checkpoint```是当前要擦除的位置，也是往后推移

每次刷盘```redo log```记录到日志文件组中，```write pos```位置就会后移更新。

每次```MySQL```加载日志文件组恢复数据时，会清空加载过的```redo log```记录，并把```checkpoint```后移更新。

```write pos```和```checkpoint```之间的还空着的部分可以用来写入新的```redo log```记录。

![img](https://linyongchao.github.io/static/img/mysql-log/10.webp)

如果```write pos```追上```checkpoint```，表示日志文件组满了，这时候不能再写入新的```redo log```记录，```MySQL```得停下来，清空一些记录，把```checkpoint```推进一下。

![img](https://linyongchao.github.io/static/img/mysql-log/11.webp)

### 小结

相信大家都知道了```redo log```的作用和它的刷盘时机、存储形式。

现在我们来思考一问题，只要每次把修改后的数据页直接刷盘不就好了，还有```redo log```什么事？它们不都是刷盘么？差别在哪里？

实际上，数据页大小是```16KB```，刷盘比较耗时，可能就修改了数据页里的几```Byte```数据，有必要把完整的数据页刷盘吗？

而且数据页刷盘是随机写，因为一个数据页对应的位置可能在硬盘文件的随机位置，所以性能很差。

如果是写```redo log```，一行记录可能就占几十```Byte```，只包含表空间号、数据页号、磁盘文件偏移量、更新值，再加上是顺序写，所以刷盘速度很快。

所以用```redo log```形式记录修改内容，性能会远远超过刷数据页的方式，这也让数据库的并发能力更强。

ps：其实内存的数据页在一定时机也会刷盘，我们把这称为页合并，讲```Buffer Pool```的时候会对这块细说
