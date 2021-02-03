---
layout: post
title:  MySQL权限问题
date:   2021-02-03 17:39:05
categories: MySQL
---

* content
{:toc}

### 问题

今天遇到了一个问题，操作数据库报错：

```
java.sql.SQLSyntaxErrorException: Access denied for user 'root'@'192.168.0.%' to database '***'
        at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:120)
        at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:97)
        at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:122)
        at com.mysql.cj.jdbc.StatementImpl.executeUpdateInternal(StatementImpl.java:1335)
        at com.mysql.cj.jdbc.StatementImpl.executeLargeUpdate(StatementImpl.java:2108)
        at com.mysql.cj.jdbc.StatementImpl.executeUpdate(StatementImpl.java:1245)
```

### 分析

这一看就是权限的问题  
首先验证用户名、密码，排除了这个原因  
然后看用户权限，看是否限制了本地访问：  

```
mysql> select user,host from mysql.user where user = 'root';
+------+-------------+
| user | host        |
+------+-------------+
| root | %           |
| root | 192.168.0.% |
| root | localhost   |
+------+-------------+
3 rows in set (0.00 sec)
```

看到```host```是既有```%```又有```192.168.0.%```，那么具体权限有什么差异呢？

```
mysql> select * from mysql.user where user = 'root';
+-------------+------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+-----------------------+-------------------+----------------+
| Host        | User | Select_priv | Insert_priv | Update_priv | Delete_priv | Create_priv | Drop_priv | Reload_priv | Shutdown_priv | Process_priv | File_priv | Grant_priv | References_priv | Index_priv | Alter_priv | Show_db_priv | Super_priv | Create_tmp_table_priv | Lock_tables_priv | Execute_priv | Repl_slave_priv | Repl_client_priv | Create_view_priv | Show_view_priv | Create_routine_priv | Alter_routine_priv | Create_user_priv | Event_priv | Trigger_priv | Create_tablespace_priv | ssl_type | ssl_cipher | x509_issuer | x509_subject | max_questions | max_updates | max_connections | max_user_connections | plugin                | authentication_string                     | password_expired | password_last_changed | password_lifetime | account_locked |
+-------------+------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+-----------------------+-------------------+----------------+
| localhost   | root | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *FF8316B344656694DF207F2C419D66ADBF1FBFB5 | N                | 2019-11-02 16:43:45   |              NULL | N              |
| %           | root | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *FF8316B344656694DF207F2C419D66ADBF1FBFB5 | N                | 2021-02-03 10:12:27   |              NULL | N              |
| 192.168.0.% | root | N           | N           | N           | N           | N           | N         | N           | N             | N            | N         | N          | N               | N          | N          | N            | N          | N                     | N                | N            | N               | N                | N                | N              | N                   | N                  | N                | N          | N            | N                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *FF8316B344656694DF207F2C419D66ADBF1FBFB5 | N                | 2021-02-03 10:22:01   |              NULL | N              |
+-------------+------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+-----------------------+-------------------+----------------+
3 rows in set (0.00 sec)
```

可以看出```192.168.0.%```其实什么权限也没有，于是就把这条记录删除了，然后```flush privileges;```  
果然问题解决了，因为删除了```192.168.0.%```之后，匹配的权限是```%```

### 后记

```192.168.0.%```这条记录是怎么入库的呢？  
和操作人员沟通后得知，其赋权使用的语句是：
```grant all privileges on `dbname`.* to 'root'@'192.168.0.%' identified by 'pwd' with grant option;```  
其实就是给了```dbname ```库所有权限，而不是希望中的给所有库所有权限  
因此建议其将语句修改为：
```grant all privileges on *.* to 'root'@'192.168.0.%' identified by 'pwd' with grant option;```

## 疑问

虽然问题解决了，但是```host```既有```%```又有```192.168.0.%```权限的时候，服务器权限选择的规则或者优先级是怎样的呢？  
猜测是根据服务器网段来的，如果服务器IP是```192.168.0.XX```则匹配```192.168.0.%```，否则匹配```%```  
尚需要理论证实。。。
