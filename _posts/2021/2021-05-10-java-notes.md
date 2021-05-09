---
layout: post
title:  Java 注释
date:   2021-05-10 00:54:05
categories: Java
---

* content
{:toc}

### 注释内跳转指定方法

来自[这里](https://blog.csdn.net/weixin_39800144/article/details/83745305)  

@link 需要{}，不需要单独一行  
@see 不需要{}，需要单独一行  

    /**
     * 校验{@link Util#check}
     *
     * @see Util#check
     */
