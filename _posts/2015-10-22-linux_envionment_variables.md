---
layout: post
title:  Linux 环境变量
date:   2015-10-22 14:49:05
categories: linux
---

* content
{:toc}

## 分类

Linux 的环境变量按照生存周期来划分可分为两类：永久生效的和临时生效的。

## 永久生效的

需要修改配置文件，常见的配置文件种类如下。

### /etc/profile

对所有用户生效；此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行；并从 /etc/profile.d 目录的配置文件中搜集shell的设置

例如：编辑/etc/profile文件，添加CLASSPATH变量

	vi /etc/profile

添加一行：

	export CLASSPATH=./JAVA_HOME/lib;$JAVA_HOME/jre/lib

修改后需要执行重新登录才能生效，也可以执行命令 source /etc/profile 来生效

### /etc/bashrc

对所有用户生效；为每一个运行bash shell的用户执行此文件。当 bash shell 被打开时，该文件被读取

编辑方法如上，不再赘述

### ~/.bash_profile

仅会对当前用户有效；每个用户都可使用该文件输入专用于自己使用的 shell 信息，当用户登录时，该文件仅仅执行一次

例如：编辑 lin 用户目录(/home/lin)下的 .bash_profile

	$ vi /home/lin/.bash_profile

添加如下内容：

	export CLASSPATH=./JAVA_HOME/lib;$JAVA_HOME/jre/lib

修改后需要执行重新登录才能生效，也可以执行命令source /etc/profile来生效

### ~/.bashrc

仅会对当前用户有效；该文件包含专用于该用户的 bash shell 的 bash 信息，当登录时以及每次打开新的 shell 时，该文件被读取

编辑方法如上，不再赘述

### 综述

~/.bashrc 等设定的变量(局部)只能继承 /etc/profile 中的变量，他们是“父子”关系

对上述文件修改，添加需要的变量，在启动一个shell（终端，terminal）时，所定义的变量均会生效的。

## 临时生效的

临时变量使用 export 命令声明即可，变量只在当前的 shell(BASH) 或其子 shell(BASH) 下是有效的，在关闭 shell 后失效，打开新 shell 要使用的话需要重新定义

在shell的命令行下直接使用 “export 变量名=变量值” 定义变量

## 环境变量的查看

1. 使用 echo 命令查看单个环境变量。例如：echo $PATH
2. 使用 env 查看所有环境变量。例如：env
3. 使用 set 查看所有本地定义的环境变量。例如：set	
4. 使用 unset 可以删除指定的环境变量。

## 常用的环境变量

1. PATH 决定了shell将到哪些目录中寻找命令或程序
2. HOME 当前用户主目录
3. HISTSIZE　历史记录数
4. LOGNAME 当前用户的登录名
5. HOSTNAME　指主机的名称
6. SHELL 当前用户Shell类型
7. LANGUGE 　语言相关的环境变量，多语言可以修改此环境变量
8. MAIL　当前用户的邮件存放目录
9. PS1　基本提示符，对于root用户是#，对于普通用户是$
