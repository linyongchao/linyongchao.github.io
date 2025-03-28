---
layout: post
title:  PostgreSQL 命令
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
	
## 索引

### 创建索引

	CREATE INDEX 【index_name】 ON 【table_name】 USING btree (【column_name】);

### 删除索引

	drop index 【index_name】;
	
## 排序

### 物理排序

	CLUSTER 【table_name】 USING 【index_name】;
	
## 导出

### 导出库

	PGPASSWORD='【password】' pg_dump -h 【ip】 -p 【port】 -U 【user_name】 -f 【file_name】 -d 【db_name】

### 导出角色

	PGPASSWORD='【password】' pg_dumpall --roles-only -h 【ip】 -p 【port】 -U 【user_name】 -f 【file_name】

## 导入

### 导入库和角色

	PGPASSWORD='【password】' psql -h 【ip】 -p 【port】 -U 【user_name】 -d 【db_name】 -f 【file_name】

## 修改密码

	ALTER ROLE 【user_name】 WITH PASSWORD '【password】';



