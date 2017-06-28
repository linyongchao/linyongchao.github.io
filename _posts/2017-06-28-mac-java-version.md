---
layout: post
title:  Mac下Java版本异常
date:   2017-06-28 09:35:05
categories: Java Mac
---

* content
{:toc}

## 问题
在命令行输入 java -version 后出现的版本是1.7

## 解决过程
1. .bash_profile  
检查.bash_profile，内部配置的是1.8  
在命令行执行 source .bash_profile 之后，再看版本是1.8
但是新窗口看版本依然是1.7
2. echo $JAVA_HOME  
在命令行查看 $JAVA_HOME 环境变量，是1.7
在命令行执行 source .bash_profile 之后，再看版本是1.8
但是新窗口看版本依然是1.7
3. .zshrc  
在网上搜索相关资料后，查看.zshrc文件
发现其内部配置的是1.7，在修改并source之后，问题解决

## 结论
Java版本错误并不是Java环境变量配置的有问题而是所使用的客户端的环境变量有问题