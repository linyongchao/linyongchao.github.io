---
layout: post
title:  Python 字符串
date:   2015-10-23 16:00:05
categories: Python
---

* content
{:toc}

### 去空格

	a = "   x y  z   "
	print a.strip()			#return "x y  z"
	print a.lstrip()		#return "x y  z   "
	print a.rstrip()		#return "   x y  z"
	print a.replace(' ','')		#return "xyz"

### 判断空

	if s.strip()=='':
	    print 's is null'
	    
	if not s.strip():
	    print 's is null'
	    
### 开头结尾

	str.endswith(X)
	str.startswith(X)
	
### 截取

来源于[这里](https://www.cnblogs.com/xunbu7/p/8074417.html)

	str = '0123456789'
	
	print str[2]		#截取第三个字符
	print str[-1]		#截取倒数第一个字符
	
	print str[:]		#截取字符串的全部字符
	print str[0:3]		#截取第一位到第三位的字符
	print str[6:]		#截取第七个字符到结尾
	print str[-3:]		#截取倒数第三位到结尾
	print str[:-3]		#截取从头开始到倒数第三个字符之前
	print str[-3:-1]	#截取倒数第三位与倒数第一位之前的字符
	
	print str[::-1]		#创造一个与原字符串顺序相反的字符串
	print str[:-5:-3]	#逆序截取，具体啥意思没搞明白？
	
结果如下：

	2
	9
	0123456789
	012
	6789
	789
	0123456
	78
	9876543210
	96
	
