---
layout: post
title:  Mac 下常用命令找不到的解决方案
date:   2021-05-26 19:09:05
categories: Mac
---

* content
{:toc}

解决方案参考[这里](https://blog.csdn.net/qq_41132565/article/details/109094161)

## 原因

```.bash_profile``` 文件中的环境变量设置错误，导致常用命令都报```command not found```

### 解决步骤

1. ```export PATH=/usr/bin:/usr/sbin:/bin:/sbin:/usr/X11R6/bin``` 执行命令后 vim 命令就可以使用了
2. ```vim .bash_profile``` 将出错的配置修改正确，保存并退出
3. ```source .bash_profile```
