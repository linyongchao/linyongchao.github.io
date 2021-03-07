---
layout: post
title:  Nginx-upload-module 配置问题
date:   2021-03-04 23:39:05
categories: Nginx
---

* content
{:toc}

## 问题及解决方案

### 问题

1. 第一次配置环境后，可以上传文件、不能上传参数
2. 第二次配置文件后，可以上传参数，不能上传文件

### 解决

	location /url/ {		--- 第二次的问题是url后面多个斜杠，去掉就可以了
	    upload_resumable on;
	    upload_state_store /tmp;
	    upload_pass @upload_handler;
	    upload_store /tmp;
	    upload_store_access user:rw group:rw all:rw;
	    upload_set_form_field "filename" $upload_file_name;
	    upload_set_form_field "filepath" $upload_tmp_path;
	    upload_aggregate_form_field "filesize" $upload_file_size;
	    proxy_connect_timeout   300s;
	    proxy_send_timeout      900s;
	    proxy_read_timeout 1800s;
	    upload_pass_form_field “^submit$|^description$”;	--- 第一次的问题是这里是中文引号，导致配置不生效
	    proxy_ignore_client_abort on;
	    upload_pass_args on;
	    client_max_body_size 5000m;
	}

## 文档

附一个靠谱的文档，原文见[这里](https://blog.osf.cn/2020/06/30/nginx-upload-module/)

### ```upload_pass```

	Syntax: upload_pass location
	Default: —
	Context: server,location

完成文件上传后，当前的POST表单内的file字段将被替换为服务器上的临时路径与其他的必须信息，并通过本参数交由下一阶段处理，可指向某个uri或upstream。

### ```upload_resumable```

	Syntax: upload_resumable on | off
	Default: upload_resumable off
	Context: main,server,location

启用断点续传。

### ```upload_store```

	Syntax: upload_store directory [level1 [level2]] ...
	Default: —
	Context: server,location

指定一个目录用于临时存储上传的文件，如指定level为1则该目录下需创建名为0-9的子目录以适配哈希化存储，目录/子目录需在Nginx启动前创建完成，Nginx不会自动创建目录/子目录。

### ```upload_state_store```

	Syntax: upload_state_store directory [level1 [level2]] ...
	Default: —
	Context: server,location

指定一个目录用于存储断点续传的临时文件，其他同upload_store。

### ```upload_store_access```

	Syntax: upload_store_access mode
	Default: upload_store_access user:rw
	Context: server,location

指定upload_store目录下临时文件的权限，无特殊需求时无需设置，使用默认值。

### ```upload_set_form_field```

	Syntax: upload_set_form_field name value
	Default: —
	Context: server,location

定义变量名用于替换POST表单内file字段，供下一阶段使用:

	$upload_field_name: HTML表单内的file字段名
	$upload_content_type: 文件MIME类型
	$upload_file_name: 不带路径的原始文件名
	$upload_tmp_path: 临时文件的绝对路径

这些变量仅在本次POST请求期间有效。
	
### ```upload_aggregate_form_field```

	Syntax: upload_aggregate_form_field name value
	Default: —
	Context: server,location

提供额外的附加在POST表单内的聚合字段，可在设置中使用Nginx变量、```upload_set_form_field```变量、以及以下所列出的变量:

	$upload_file_md5: 上传文件的MD5哈希值(小写)
	$upload_file_md5_uc: 上传文件的MD5哈希值(大写)
	$upload_file_sha1: 上传文件的SHA1哈希值(小写)
	$upload_file_sha1_uc: 上传文件的SHA1哈希值(大写)
	$upload_file_sha256: 上传文件的SHA256哈希值(小写)
	$upload_file_sha256_uc: 上传文件的SHA256哈希值(大写)
	$upload_file_sha512: 上传文件的SHA512哈希值(小写)
	$upload_file_sha512_uc: 上传文件的SHA512哈希值(大写)
	$upload_file_crc32: 上传文件的CRC32校验码(十六进制)
	$upload_file_size: 上传文件的大小(bytes)
	$upload_file_number: 上传文件的序号(从1开始)
	
这些变量仅在本次POST请求期间有效。

注意: 除去```$upload_file_size```, ```$upload_file_number```这两个变量外，其他的变量一旦被设置均会消耗额外的CPU资源用于计算哈希值，同时设置一种哈希算法的大小写变量仅进行一次哈希计算。

### ```upload_pass_form_field```

	Syntax: upload_pass_form_field regex
	Default: —
	Context: server,location

默认情况下，本模块并不会将file类型字段以外的input字段发送给下一阶段处理，除非在此显式声明字段名称(非类型)。

使用范例(PCRE支持):

	upload_pass_form_field "^submit$|^test$";

使用范例(无PCRE):

	upload_pass_form_field "submit";
	upload_pass_form_field "test";

### ```upload_cleanup```

	Syntax: upload_cleanup status/range ...
	Default: —
	Context: server,location

指定一系列HTTP状态码，范围201-599，可以用减号指定范围，当下一阶段处理完毕返回的HTTP状态码与此处匹配时则删除当前POST请求上传的临时文件。

使用范例:

	upload_cleanup 400 404 499 500-505;

### ```upload_buffer_size```

	Syntax: upload_buffer_size size
	Default: size of memory page in bytes
	Context: server,location

写缓冲区尺寸，默认为一个内存页的大小(4k)，如果希望减少系统呼叫(syscall write)的次数，可增加该值。

### ```upload_max_part_header_len```

	Syntax: upload_max_part_header_len size
	Default: 512
	Context: server,location

设置part header的长度，绝大多数时候保持默认值即可

	part header长这样:
	------WebKitFormBoundarydjT6Js1Pew3Fxmyb
	或者这样:
	--------------------------858745820d5862d4
	如果part header超长,你会在错误日志看到下面这行提示:
	*53 part header is too long
	
### ```upload_max_file_size```

	Syntax: upload_max_file_size size
	Default: 0
	Context: main,server,location

设定单个文件的尺寸限制，默认为0，不限制文件大小。该设置为软限制，即使文件大小超限Nginx也不会中断与报错，超限文件仅被模块忽略，较而言之```client_max_body_size``` 是Nginx声明的硬限制，超限会导致上传中断与报错。

### ```upload_limit_rate```

	Syntax: upload_limit_rate rate
	Default: 0
	Context: main,server,location

上传速率限制，默认为0，不限制上传速度。

### ```upload_max_output_body_len```

	Syntax: upload_max_output_body_len size
	Default: 100k
	Context: main,server,location

定义送至下一阶段的POST请求内body最大尺寸，默认为100k，如果超限会收到错误提示error 413 (Request entity too large)。设置为0时不限制。

### ```upload_tame_arrays```

	Syntax: upload_tame_arrays on | off
	Default: off
	Context: main,server,location

是否删除文件字段名称中的方括号。(当你的后端是PHP时，需要打开该选项)。

### ```upload_pass_args```

	Syntax: upload_pass_args on | off
	Default: off
	Context: main,server,location

转发查询参数至upload_pass指定的location，对命名location无效，如@test。

## 采坑全程

### 背景

最近要做一个断点续传的功能，同事告诉我：别的项目有现成的，不要开发了，直接拿过来用  
我看了看别的项目，是用的```Nginx-upload-module```，就让运维帮忙部署环境  
运维环境部署好了，我开始代码调试

### 过程

1. 傍晚调试时发现文件可以上传成功，但是参数没有提交。换着各种方式提交参数，都不行。我就准备去看看怎么配置的 nginx，结果发现配置的文件被删了。。。
2. 第二天让运维再配置一次，这次参数可以提交了，但是文件上传不了。。。
3. 去找运维，运维说没有问题。。。

好吧，那就没有办法了，我只好自己去看看这个插件了  
结果一番操作，有价值的能解决问题的几乎没有，包括我怀疑的```upload_pass_form_field```字段后面为啥是中文引号，结果十篇文章中有八篇也是中文。。。  
后来找的附录中的这篇，里面是英文引号，我觉得还是很靠谱的，就附录一下  
至于url，真的没有找的一篇文章说这个问题，最后是比对其他项目的配置文件，发现有差异，就改了一下，结果问题就解决了，囧o(╯□╰)o