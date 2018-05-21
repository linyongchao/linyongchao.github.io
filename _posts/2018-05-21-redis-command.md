---
layout: post
title:  Redis 命令
date:   2018-05-21 21:08:54
categories: Redis
---

* content
{:toc}

### 批量删除

	./redis-cli keys "test:*" | xargs ./redis-cli del
