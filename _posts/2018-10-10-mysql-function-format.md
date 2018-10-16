---
layout: post
title:  FORMAT
date:   2018-10-10 16:11:54
categories: MySQL
---

* content
{:toc}

## 前言

遇到一个需求：将数字四舍五入保留两位小数格式化  
查到一篇[文章](https://www.jb51.net/article/44378.htm)讲了三种方案：  

1. FORMAT

		MariaDB [(none)]> SELECT FORMAT(12345.6789,2);
		+----------------------+
		| FORMAT(12345.6789,2) |
		+----------------------+
		| 12,345.68            |
		+----------------------+

	没有达到预期结果，因为不想以逗号分隔

2. TRUNCATE

		MariaDB [(none)]> SELECT TRUNCATE(12345.6789,2);
		+------------------------+
		| TRUNCATE(12345.6789,2) |
		+------------------------+
		|               12345.67 |
		+------------------------+
	
	没有达到预期结果，因为没有四舍五入

3. CONVERT

		MariaDB [(none)]> SELECT CONVERT(12345.6789,decimal(10,2));
		+-----------------------------------+
		| CONVERT(12345.6789,decimal(10,2)) |
		+-----------------------------------+
		|                          12345.68 |
		+-----------------------------------+
		
	达到预期结果

我对这个结论感觉很奇怪，主要是```FORMAT```的结果只能以逗号分隔让我感觉很惊讶，于是就去查了查文档，发现了一个很有意思的事情，特记录如下

## FORMAT

### 定义

官网对```FORMAT```函数的定义见[这里](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_format)  
全文如下：  

	FORMAT(X,D[,locale])
	
	Formats the number X to a format like '#,###,###.##', rounded to D decimal places, and returns the result as a string. If D is 0, the result has no decimal point or fractional part.
	
	The optional third parameter enables a locale to be specified to be used for the result number's decimal point, thousands separator, and grouping between separators. Permissible locale values are the same as the legal values for the lc_time_names system variable (see Section 10.15, “MySQL Server Locale Support”). If no locale is specified, the default is 'en_US'.
	
翻译如下：  

	FORMAT(X,D[,locale])
	
	将数字 X 格式化成 '#,###,###.##' 的形式，四舍五入保留 D 位小数，并且返回值是一个字符串。
	如果 D 为 0，则返回值没有小数点和小数部分。
	
	可选的第三个参数用于指定返回值的小数点、千分号和分隔符的分组设置。
	允许的值和系统变量 lc_time_names 相同。如果未指定则默认 en_US。
	
也就是说，```FORMAT```函数并不是通常认为的两个参数，其可选的第三个参数可以指定分隔符  
根据官网在[Section 10.15, “MySQL Server Locale Support”](https://dev.mysql.com/doc/refman/8.0/en/locale-support.html)给出的链接，我们继续探究

### ```lc_time_names```
	
在跳转页面可以发现如下内容：

	lc_time_names may be set to any of the following locale values. The set of locales supported by MySQL may differ from those supported by your operating system.
	
翻译如下：

	lc_time_names 可以设置为以下任何区域值。
	MySQL支持的区域设置可能与您的操作系统支持的区域设置不同。
	
下方的表格共计 111 个区域值，我们将其取出

### 测试

依次用区域值格式化数字```123456789.123456```  
执行命令举例如下：

	MariaDB [(none)]> select FORMAT(123456789.123456, 4,'zh_CN');
	+-------------------------------------+
	| FORMAT(123456789.123456, 4,'zh_CN') |
	+-------------------------------------+
	| 123,456,789.1235                    |
	+-------------------------------------+
	
然后我们将格式化后的内容做统计，结论如下：

	+-------------------+-----+
	| format            | num |
	+-------------------+-----+
	| 12,34,56,789.1235 |   3 |
	| 123 456 789,1235  |   1 |
	| 123 456 789,1235  |   9 |
	| 123'456'789,1235  |   2 |
	| 123'456'789.1235  |   1 |
	| 123,456,789.1235  |  37 |
	| 123.456.789,1235  |  20 |
	| 123456789,1235    |  26 |
	| 123456789.1235    |  12 |
	+-------------------+-----+
	9 rows in set (0.00 sec)
	
可见，可以满足我们格式化需求的区域值有12个，分别如下：

	+----------+----------------+
	| language | format         |
	+----------+----------------+
	| ar_SA    | 123456789.1235 |
	| es_CR    | 123456789.1235 |
	| es_DO    | 123456789.1235 |
	| es_GT    | 123456789.1235 |
	| es_HN    | 123456789.1235 |
	| es_MX    | 123456789.1235 |
	| es_NI    | 123456789.1235 |
	| es_PA    | 123456789.1235 |
	| es_PE    | 123456789.1235 |
	| es_PR    | 123456789.1235 |
	| es_SV    | 123456789.1235 |
	| sr_RS    | 123456789.1235 |
	+----------+----------------+
	12 rows in set (0.00 sec)
	
挑一个验证一下：

	MariaDB [(none)]> select FORMAT(123456789.123456, 4,'ar_SA');
	+-------------------------------------+
	| FORMAT(123456789.123456, 4,'ar_SA') |
	+-------------------------------------+
	| 123456789.1235                      |
	+-------------------------------------+

## 结论

1. ```FORMAT```的结果支持 9 种分隔方式  
2. 最终用```CONVERT```实现的需求  
┓( ´∀` )┏

