---
layout: post
title:  PostgreSQL ALTER
date:   2017-06-29 19:05:05
categories: PostgreSQL
---

* content
{:toc}

### 增加列 

	ALTER TABLE table_name ADD column_name datatype;
	
### 删除列  

	ALTER TABLE table_name DROP  column_name; 
	
### 改列的数据类型  

	ALTER TABLE table_name ALTER  column_name TYPE datatype;
	
### 表重命名  

	ALTER TABLE table_name RENAME TO new_name;

### 列重命名
  
	ALTER TABLE table_name RENAME column_name to new_column_name;

### 设置NOT NULL
  
	ALTER TABLE table_name ALTER column_name {SET|DROP} NOT NULL; 

### 设置DEFAULT

	ALTER TABLE table_name ALTER column_name SET DEFAULT expression;
	