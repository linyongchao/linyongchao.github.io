---
layout: post
title:  APNS 中遇到的问题
date:   2021-03-25 17:49:05
categories: APNS
---

* content
{:toc}

### ExpiredProviderToken

解决方案来自[这里](https://www.jianshu.com/p/d8dba6c2c07a)  
推送时返回：

	{
		"reason": "ExpiredProviderToken"
	}
	
问题原因：为了保证安全，APNs要求定期更新token，时间间隔为1小时；如果APNs发现当前的时间戳与iat值中的时间戳相比，大于一个小时，那么APNs会拒绝推送消息，并返回ExpiredProviderToken (403)错误。  
解决方案：同步服务器时间
