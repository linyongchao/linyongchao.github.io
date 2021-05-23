---
layout: post
title:  PostgreSQL ALTER
date:   2017-07-18 19:05:05
categories: DB
---

* content
{:toc}

## 命令行

### 查看所有库

	\l

### 查看所有表

	\dt
	
### 查看表结构

	\d [table]

## 表

### 复制表

	CREATE TABLE table_name_A AS (SELECT * FROM table_name_B);

### 表重命名  

	ALTER TABLE table_name RENAME TO new_name;
	
## 列

### 增加列 

	ALTER TABLE table_name ADD column_name datatype NOT NULL DEFAULT 1;
	
### 删除列  

	ALTER TABLE table_name DROP  column_name; 
	
### 列重命名
  
	ALTER TABLE table_name RENAME column_name to new_column_name;
	
### 列类型修改  

	ALTER TABLE table_name ALTER  column_name TYPE datatype;
	
### 设置 NOT NULL
  
	ALTER TABLE table_name ALTER column_name {SET|DROP} NOT NULL; 

### 设置 DEFAULT

	ALTER TABLE table_name ALTER column_name SET DEFAULT expression;
	
### 设置 COMMENT

	COMMENT ON COLUMN table_name.column_name IS 'desc';
	