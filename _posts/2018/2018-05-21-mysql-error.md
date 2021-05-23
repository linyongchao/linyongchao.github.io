---
layout: post
title:  MySQL 错误
date:   2018-05-21 20:08:54
categories: MySQL
---

* content
{:toc}

## JDBC
### Before start of result set

异常：```java.sql.SQLException: Before start of result set```  
解决：调用```rs.getString();```前一定要加上```rs.next();```   
原因：ResultSet对象代表SQL语句执行的结果集，维护指向其当前数据行的光标。每调用一次next()方法，光标向下移动一行。最初它位于第一行之前，因此第一次调用next()应把光标置于第一行上，使它成为当前行。随着每次调用next()将导致光标向下移动一行。在ResultSe对象及其t父辈Statement对象关闭之前，光标一直保持有效。   
参考：[这里](https://blog.csdn.net/killua_hzl/article/details/6073618)
