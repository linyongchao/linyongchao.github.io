---
layout: post
title:  MySQL 存在则更新，不存在则新增
date:   2019-07-21 22:06:54
categories: MySQL
---

* content
{:toc}

参考 [这里](https://www.cnblogs.com/Eric-zhao/p/6655994.html) 和 [这里](https://blog.csdn.net/eleanoryss/article/details/82997899)

过程分两步：

1. 将判断数据是否存在的字段设置为唯一索引
2. 通过 ```INSERT INTO …… ON DUPLICATE KEY UPDATE ……``` 语句实现目标

实践：

在 ```tb_user``` 表先有四条记录的情况下，先多次执行语句1，再多次执行语句2，再多次执行语句3，得到结果如表格：

	INSERT INTO tb_user (login_name,user_name,sex,age) VALUES ('li','小李',1,24) ON DUPLICATE KEY UPDATE user_name = '小李' , sex = 1 , age = 24;
	INSERT INTO tb_user (login_name,user_name,sex,age) VALUES ('zhou','小周',1,26) ON DUPLICATE KEY UPDATE user_name = '小周' , sex = 1 , age = 26;
	INSERT INTO tb_user (login_name,user_name,sex,age) VALUES ('wu','小吴',1,28) ON DUPLICATE KEY UPDATE user_name = '小吴' , sex = 1 , age = 28;

|id|login_name|user_name|sex|age|
|:---:|:---:|:---:|:---:|:---:|
|1|zhao|小赵|1|18|
|2|qian|小钱|1|20|
|3|sun|小孙|1|22|
|4|li|小李|1|24|
|14|zhou|小周|1|26|
|21|wu|小吴|1|28|

可以看到自增主键不连续了  
经过多次验证后可以确定，语句每执行一次，表 AUTO_INCREMENT 会加 1，原因见参考文章的[后一篇](https://blog.csdn.net/eleanoryss/article/details/82997899)

结论：别用。可以修改为先根据主键或者唯一索引修改，修改记录为0则再新增