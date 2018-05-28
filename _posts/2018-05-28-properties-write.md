---
layout: post
title:  Properties Write
date:   2018-05-28 22:08:54
categories: Java
---

* content
{:toc}

之前配置文件只是读取，第一次看到写配置文件，所以记录如下：

### 准备
新建文件```test.properties```，内容如下：

	name=zhangsan

### 写文件

	String path = "/Users/lin/test.properties";
	try {
	    FileReader reader = new FileReader(path);
	    Properties p = new Properties();
	    p.load(reader);
	
	    System.out.println(p);
	
	    p.setProperty("age", "18");
	    p.setProperty("address", "中国");
	    p.setProperty("sex", "男");
	
	    FileWriter writer = new FileWriter(path);
	    p.store(writer, "add properties");
	
	    System.out.println(p);
	
	    reader.close();
	    writer.close();
	} catch (Exception e) {
	    e.printStackTrace();
	}

### 结果

输出结果：

	{name=zhangsan}
	{address=中国, age=18, name=zhangsan, sex=男}

```test.properties```，内容如下：
	
	#add properties
	#Mon May 28 16:38:52 CST 2018
	address=中国
	age=18
	name=zhangsan
	sex=男
 
### 备注

p.store 方法的第二个参数 comments 最好使用英文  
我第一次测试使用中文“新增属性”，结果被转义了：
	
	#\u65B0\u589E\u5C5E\u6027
	#Mon May 28 16:36:19 CST 2018
