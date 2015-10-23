---
layout: post
title:  Python 去除字符串空格
date:   2015-10-23 16:00:05
categories: Python
---

* content
{:toc}

	a = "   x y  z   "
	print a.strip()			#return "x y  z"
	print a.lstrip()		#return "x y  z   "
	print a.rstrip()		#return "   x y  z"
	print a.replace(' ','')		#return "xyz"
