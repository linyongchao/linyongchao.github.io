---
layout: post
title:  resultSet.getLong 精度丢失问题
date:   2021-02-03 18:39:05
categories: Java
---

* content
{:toc}

### 问题

昨天遇到了一个问题，表现为：发了一次请求，结果循环处理了。。。  
分析后发现整个流程是这样的：  

1. 发一次请求，在数据库存一条记录
2. 定时任务从数据库取数据，进行处理，最后删除

问题就发生在删除时没有删除成功，导致定时任务一直取一直有数据一直处理  
没有删除成功的原因是：查询时```resultSet.getLong```精度丢失，导致主键和数据库不一致了，所以根据主键删除时失败

### 验证

在代码中增加日志：

![jdbc_code](https://linyongchao.github.io/static/img/jdbc_code.png)

查看日志：

![jdbc_log](https://linyongchao.github.io/static/img/jdbc_log.png)

### 解决

先```getString```再```parseLong```  
```Long.parseLong(resultSet.getString("id"))```
