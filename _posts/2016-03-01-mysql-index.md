---
layout: post
title:  "MySQL 索引类型"
date:   2016-02-24 19:57:05
categories: MySQL
---

* content
{:toc}

1. primary：唯一索引，不允许为null。

2. key：普通非唯一索引。

3. unique：表示唯一的，不允许重复的索引，可以为null。

4. fulltext: 表示全文搜索的索引。 FULLTEXT用于搜索很长一篇文章的时候，效果最好。用在比较短的文本，如果就一两行字的，普通的INDEX 也可以。

5. spatial：空间索引。
