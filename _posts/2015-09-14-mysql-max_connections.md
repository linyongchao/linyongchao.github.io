---
layout: post
title:  "MySQL 修改最大连接数"
date:   2015-09-14 19:47:05
categories: MySQL
---

* content
{:toc}

## 原因

要将 MySQL 的最大连接数由 100 修改为 1000，主要方法有两个：

## 方法一

1. 进入 MySQL 安装目录，打开 MySQL 配置文件 my.ini 或 my.cnf
2. 查找 max_connections=100 修改为 max_connections=1000
3. 重启 MySQL 即可

## 方法二

1. 进入：mysql -uusername -ppassword
2. 设置：mysql> set GLOBAL max_connections=1000;
3. 查看：mysql> show variables like 'max_connections';

## 注意

方法二重启 MySQL 后失效，因为重启时是从配置文件中读取配置信息的
