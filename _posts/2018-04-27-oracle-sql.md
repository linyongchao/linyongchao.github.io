---
layout: post
title:  Oracle SQL
date:   2018-04-27 16:25:54
categories: Oracle
---

* content
{:toc}

## 分页

	SELECT *
	FROM (
		SELECT ROWNUM AS rowno, t.*	
		FROM emp t
		WHERE ROWNUM <= 20
		) A
	WHERE A.rowno >= 10;
