---
layout: post
title:  Python时间格式化
date:   2017-04-12 13:53:05
categories: Python
---

* content
{:toc}

time.strftime 可以实现时间格式化  
time.strptime 可以实现字符串转时间

	>>> import time
	>>> time.localtime()
	time.struct_time(tm_year=2017, tm_mon=4, tm_mday=12, tm_hour=13, tm_min=44, tm_sec=57, tm_wday=2, tm_yday=102, tm_isdst=0)
	>>> print time.strftime('%Y-%m-%d',time.localtime())
	2017-04-12
	>>> time.strptime('2017-04-12','%Y-%m-%d')
	time.struct_time(tm_year=2017, tm_mon=4, tm_mday=12, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=2, tm_yday=102, tm_isdst=-1)
	>>>

格式化符号

	%a 英文星期简写
	%A 英文星期的完全
	%b 英文月份的简写
	%B 英文月份的完全
	%c 显示本地日期时间
	%d 日期，取1-31
	%H 小时， 0-23
	%I 小时， 0-12 
	%m 月， 01 -12
	%M 分钟，1-59
	%j 年中当天的天数
	%w 显示今天是星期几
	%W 第几周
	%x 当天日期
	%X 本地的当天时间
	%y 年份 00-99间
	%Y 年份的完整拼写