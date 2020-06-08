---
layout: post
title:  MyISAM 和 InnoDB
date:   2020-06-08 18:59:54
categories: Java
---

* content
{:toc}

本文参考：[这里](https://www.zhihu.com/question/20596402)、[这里](https://www.cnblogs.com/ConnorShip/p/10024498.html)

### 区别

1. InnoDB 支持事务，MyISAM 不支持事务。这是 MySQL 将默认存储引擎从 MyISAM 换到 InnoDB 的重要原因。
2. InnoDB 支持外键，MyISAM 不支持外键。对一个包含外键的 InnoDB 表转换为 MyISAM 会失败。
3. InnoDB 是聚集索引，MyISAM 是非聚集索引。聚集索引的文件放在主键索引的叶子节点上，所以 InnoDB 必须要有主键，通过主键索引效率很高。MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。
4. InnoDB 不保存表的具体行数。执行 ```SELECT COUNT(*) FROM table``` 时需要全表扫描。MyISAM 用一个变量保存了整个表的行数，因此统计全表行数速度很快。
5. InnoDB 最小的锁粒度是行锁，MyISAM 最小的锁粒度是表锁。
6. InnoDB 支持 MVCC，MyISAM 不支持。
7. InnoDB 5.6 之前不支持全文索引（5.6及之后支持），MyISAM支持。

### 选择

* 是否需要支持事务。如果要支持事务，选择 InnoDB
* 如果绝大多数是读查询操作，选择 MyISAM；如果读写均很频繁，则选择 InnoDB
* 系统崩溃后，MyISAM 恢复更为困难，如果不能接受，则选择 InnoDB

