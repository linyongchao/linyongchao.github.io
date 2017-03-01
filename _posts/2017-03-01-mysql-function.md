---
layout: post
title:  "MySQL 函数"
date:   2017-03-01 19:57:05
categories: MySQL
---

* content
{:toc}

## CAST

转类型

```
CAST(expr AS type)

type取值如下：
BINARY[(N)]
CHAR[(N)]
DATE
DATETIME
DECIMAL
SIGNED [INTEGER]
TIME
UNSIGNED [INTEGER]
```
eg:

```
SELECT CAST(123 AS CHAR)	#强制转成字符串
```

## GROUP_CONCAT

拼接

```
GROUP_CONCAT(
[DISTINCT] expr [,expr ...]
[ORDER BY {unsigned_integer | col_name | expr} [ASC | DESC] [,col_name ...]]
[SEPARATOR str_val])
```

eg:

```
select GROUP_CONCAT(id) from A;
select GROUP_CONCAT(id ORDER BY id DESC) from A;
select GROUP_CONCAT(id SEPARATOR ';') from A;
```

注意：  
1. int字段的连接陷阱  
连接的字段如果是int型，一定要转换成char再拼接  
否则返回的将不是一个逗号隔开的串，而是byte[]  
2. 长度陷阱  
GROUP_CONCAT连接字段的时候是有长度限制的，并不是有多少连多少  
使用group_concat_max_len系统变量，可以设置允许的最大长度：  
SET [SESSION | GLOBAL] group_concat_max_len = val  
若已经设置了最大长度， 则结果被截至这个最大长度