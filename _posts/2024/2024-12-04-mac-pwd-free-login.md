---
layout: post
title:  Mac下配置免密登录
date:   2024-12-04 16:19:05
categories: Mac
---

* content
{:toc}

## 综述

```
cd ~/.ssh
ssh-keygen -t rsa -f [秘钥文件名，不指定可能会覆盖默认的id_rsa]
ssh-copy-id -i [秘钥文件名] [用户名]@[IP地址]
ssh-add -K [秘钥文件名]
```

## 详解

### 进入文件夹

```
cd ~/.ssh
```

接下来所有操作都在该文件夹下进行

### 生成秘钥

```
ssh-keygen -t rsa -f [秘钥文件名，不指定可能会覆盖默认的id_rsa]
```

如果不指定秘钥文件名，则默认生成的文件名称为```id_rsa```  
如果```id_rsa```文件已经存在，则会覆盖，所以尽量指定秘钥文件名称

比如为33服务器生成的秘钥，命名为：pwd33，则命令为：

```
ssh-keygen -t rsa -f pwd33
```

### 上传秘钥

```
ssh-copy-id -i [秘钥文件名] [用户名]@[IP地址]
```

命令示例：

```
ssh-copy-id -i pwd33 root@192.168.1.33
```

### 缓存秘钥

```
ssh-add -K [秘钥文件名]
```

```ssh-add```命令是把密钥添加到```ssh-agent```的高速缓存中  

Mac下这一步必须进行，否则无法免密登录，命令示例：

```
ssh-add -K pwd33
```

至此，所有步骤结束
