---
layout: post
title:  EXPLAIN 分析 SQL
date:   2020-05-06 18:59:54
categories: DB
---

* content
{:toc}

本文参考：[这里](https://www.linuxidc.com/Linux/2018-08/153354.htm)、[这里](https://www.cnblogs.com/deverz/p/11066043.html)

由下可见，EXPLAIN 分析结果由 id、select_type、table、partitions、type、possible_keys、key、key_len、ref、rows、filtered、Extra 等列组成，我们逐个解析列的含义。

	mysql> EXPLAIN SELECT 1;
	+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
	| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
	+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
	|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | No tables used |
	+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
	1 row in set, 1 warning (0.00 sec)

### id



### select_type

表示 SELECT 类型，其取值有：

|类型|说明|
|:-:|:-:|
|SIMPLE|简单表，不使用表连接或子查询|
|PRIMARY|主查询，即外层的查询|
|UNION|UNION中的第二个或者后面的查询语句|
|SUBQUERY|子查询中的第一个|

### table

输出结果集的表（表别名）

### partitions



### type

表示MySQL在表中找到所需行的方式，或者叫访问类型。这是最重要的字段之一，一般来说，得保证查询至少达到range级别，最好能达到ref。  
常见访问类型如下，从上到下，性能由差到最好：

|类型|含义|可能的场景|
|:-:|:-:|:-:|
|ALL|全表扫描|一般是没有where条件或者where条件没有使用索引|
|index|索引全扫描|遍历整个索引来查询匹配行，并不会扫描表，一般是查询的字段都有索引|
|range|索引范围扫描|where条件带<、<=、>、>=、between等操作且有索引|
|ref|非唯一索引扫描|使用非唯一索引或唯一索引的前缀扫描或者join操作的非全表扫描的表|
|eq_ref|唯一索引扫描|索引是唯一索引，一般出现在多表连接时使用主键或者唯一索引作为关联条件|
|const,system|单表最多有一个匹配行|出现在根据主键primary key或者 唯一索引 unique index 进行的查询|
|NULL|不用扫描表或索引|不用访问表或者索引，直接就能够得到结果|

### possible_keys

表示查询时可能使用的索引。如果为空，表示没有可能应用的索引。

### key

实际使用的索引。如果为NULL，则没有使用索引。  
可以在SELECT语句中使用FORCE INDEX（index_name）来强制使用一个索引或者用IGNORE INDEX（index_name）来强制忽略索引。

### key_len

使用索引字段的长度。在不损失精确性的情况下，长度越短越好。

### ref

使用哪个列或常数与key一起从表中选择行

### rows

扫描行的数量

### filtered

存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例(百分比)

### Extra

执行情况的说明和描述，包含不适合在其他列中显示但是对执行计划非常重要的额外信息。  
见到 Using temporary 和 Using filesort，就意味着MySQL根本不能使用索引，结果是检索会很慢，需要优化sql了。  
主要类型如下：

|类型|含义|
|:-:|:-:|
|Using Index|表示索引覆盖，不会回表查询|
|Using Where|表示进行了回表查询|
|Using temporary|用到临时表去处理当前的查询|
|Using Index Condition|表示进行了ICP优化|
|Using Flesort|表示MySQL需额外排序操作, 不能通过索引顺序达到排序效果|

#### ICP

MySQL5.6引入了Index Condition Pushdown（ICP）的特性，进一步优化了查询。Pushdown表示操作下放，某些情况下的条件过滤操作下放到存储引擎。

比如SQL：

	EXPLAIN SELECT * FROM rental WHERE rental_date='2005-05-25' AND customer_id>=300 AND customer_id<=400;

* 在5.6版本之前：
优化器首先使用复合索引idx_rental_date过滤出符合条件rental_date='2005-05-25'的记录，然后根据复合索引idx_rental_date回表获取记录，最终根据条件customer_id>=300 AND customer_id<=400过滤出最后的查询结果（在服务层完成）。

* 在5.6版本之后：
MySQL使用了ICP来进一步优化查询，在检索的时候，把条件customer_id>=300 AND customer_id<=400也推到存储引擎层完成过滤，这样能够降低不必要的IO访问。Extra为Using index condition就表示使用了ICP优化。





