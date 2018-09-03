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