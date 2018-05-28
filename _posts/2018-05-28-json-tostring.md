---
layout: post
title:  JSONObject.toString() 的性能问题
date:   2018-05-28 21:08:54
categories: Error
---

* content
{:toc}

## 问题
测试反馈：生成部门树很慢  
我看了一下，其勾选的部门比较多，共计1512个  
其用时特别长，日志显示：组装部门树用时50238毫秒，平均每个33毫秒  
检查代码发现有一句：```System.out.println(obj.toString());```  
obj是JSONObject对象  
将该语句删除后，日志显示：组装部门树用时237毫秒，平均每个0.16毫秒  
问题解决
## 后续
问题复盘，想到一点，也许不是JSONObject.toString()导致的性能问题，说不定是输出呢  
于是测试如下
### 纯输出
	
	long start = System.currentTimeMillis();
	for (int i = 0; i < 2000; i++) {
	    System.out.println(i);
	}
	long end = System.currentTimeMillis();
	System.out.println("全部用时" + (end - start) + "毫秒");
        
全部用时23毫秒  
全部用时21毫秒  
全部用时23毫秒  
全部用时22毫秒  
全部用时22毫秒  
输出2000次，平均总用时22.2毫秒  
### Json 全部
	
	long start = System.currentTimeMillis();
	JSONObject json = new JSONObject();
	for (int i = 0; i < 2000; i++) {
	    json.put(i, i);
	    System.out.println(json.toString());
	}
	long end = System.currentTimeMillis();
	result = "全部用时" + (end - start) + "毫秒";
        
全部用时2072毫秒  
全部用时1976毫秒  
全部用时1979毫秒  
全部用时2161毫秒  
全部用时1769毫秒  
输出2000次，平均总用时1991.4毫秒  
### Json 分别测试
上例中无法验证是put方法影响性能还是toString方法影响性能，所以又追加一次测试：

	long allStart = System.currentTimeMillis();
	JSONObject json = new JSONObject();
	long putAll = 0L;
	long printAll = 0L;
	for (int i = 0; i < 2000; i++) {
	    long start = System.currentTimeMillis();
	    json.put(i, i);
	    long middle = System.currentTimeMillis();
	    putAll += middle - start;
	    System.out.println(json.toString());
	    long end = System.currentTimeMillis();
	    printAll += end - middle;
	}
	System.out.println("put用时" + putAll + "毫秒");
	System.out.println("打印用时" + printAll + "毫秒");
	long allEnd = System.currentTimeMillis();
	System.out.println("全部用时" + (allEnd - allStart) + "毫秒");
        
put用时18毫秒  
打印用时1832毫秒  
全部用时1852毫秒  

put用时22毫秒  
打印用时1862毫秒  
全部用时1889毫秒  

put用时14毫秒  
打印用时1881毫秒  
全部用时1895毫秒  

put用时17毫秒  
打印用时1760毫秒  
全部用时1779毫秒  

put用时27毫秒  
打印用时2234毫秒  
全部用时2264毫秒  

将打印用时减去纯输出用时，基本可以看做 toString 的用时，和 put 比较起来，几乎是百倍的差距

## 结论
1. 相对于 put 方法，JSONObject 的 toString 方法还是很慢的，应该尽量避免频繁的 toString 操作  
2. json 越复杂，预计 toString 用时越长（根据部门 toString 平均每个用时33毫秒，测试平均每个用时 1 毫秒推测）
3. 调试信息该删除就要删除！！！