---
layout: post
title:  Oracle SQL
date:   2018-04-27 16:25:54
categories: Oracle
---

* content
{:toc}

### 分页

	SELECT *
	FROM (
		SELECT ROWNUM AS rowno, t.*	
		FROM emp t
		WHERE ROWNUM <= 20
		) A
	WHERE A.rowno >= 10;
	
### 查看所有数据库

	select * from v$database;
	select name from v$database;
	SELECT Total.name "Tablespace Name",Free_space, (total_space-Free_space) Used_space, total_space FROM (select tablespace_name, sum(bytes/1024/1024) Free_Space from sys.dba_free_space group by tablespace_name) Free,(select b.name, sum(bytes/1024/1024) TOTAL_SPACE from sys.v_$datafile a, sys.v_$tablespace B where a.ts# = b.ts# group by b.name) Total WHERE Free.Tablespace_name = Total.name;

### 查看所有数据库实例

	select * from v$instance;
	select instance_name from v$instance;
	show parameter instance_name
	
### 查看当前库的所有表

	select TABLE_NAME from all_tables;
	select table_name from dba_tables
	
### 查看当前库的所有用户

	select * from all_users;
	
### 建表

	create table test(
		id int not null,
		name varchar(8) not null,
		gender varchar2(2) not null,
		age int not null,
		address varchar2(20) not null,
		regdata date
	);

