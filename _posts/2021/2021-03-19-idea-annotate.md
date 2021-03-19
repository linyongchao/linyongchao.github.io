---
layout: post
title:  Annotate 置灰问题解决方案
date:   2021-03-19 18:39:05
categories: IDEA
---

* content
{:toc}

本文参考[这里](http://cn.voidcc.com/question/p-twypxcgb-bnr.html)和[这里](https://blog.csdn.net/sanyuedexuanlv/article/details/78509721)  
今天解决了困扰多时的问题，真开心

## 问题

如图可见，Annotate按钮是置灰的，但是，其他项目是正常的

![memory-layout](https://linyongchao.github.io/static/img/idea/annotate1.png)

## 解决步骤

### 第一步

根据第一篇文章，打开```File > Settings > Version Control```  
发现有问题的项目在```Unregistered roots:```下

但其提示的的开启 VCS 执行不了，因为没有该菜单

![memory-layout](https://linyongchao.github.io/static/img/idea/annotate2.png)

原文如下：

```
如果检查File > Settings > Version Control并看到当前的项目将在“未注册的根源上市“，去（在菜单栏上）VCS > Enable Version Control Integration。
```

### 第二步

根据第二篇文章，选中项目，点击加号，最后保存即可  

![memory-layout](https://linyongchao.github.io/static/img/idea/annotate3.png)

原文如下：

```
Click the "Add root" option. 
```
