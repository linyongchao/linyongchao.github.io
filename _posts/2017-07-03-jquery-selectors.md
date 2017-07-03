---
layout: post
title:  jQuery选择器总结
date:   2017-07-03 18:05:05
categories: JS
---

* content
{:toc}

jQuery 的选择器可谓强大无比，这里简单地总结一下常用的元素查找方法
 
### 基本选择器
1. ID选择器   
$("#my") 选择id值等于my的元素，id值不能重复，所以得到的是唯一的元素  
2. 元素选择器  
$("div") 选择所有的div标签元素，返回div元素数组 
3. 样式选择器  
$(".myClass") 选择使用myClass类的css的所有元素  
4. 联合选择  
$("#myELement,div,.myclass") 
5. 全部选择  
$("*") 选择文档中的所有的元素
 
### 层叠选择器 
	$("form input")        选择所有的form元素中的input元素 
	$("#main > *")         选择id值为main的所有的子元素 
	$("label + input")     选择label标签后面直接跟一个input标签的所有input标签元素 
	$("#prev ~ div")       同胞选择器，该选择器返回的为id为prev的标签元素的所有的属于同一个父元素的div标签 
 
### 基本过滤选择器
	$("tr:first")             选择所有tr元素的第一个 
	$("tr:last")              选择所有tr元素的最后一个 
	$("input:not(:checked)")  过滤掉：checked的选择器的所有的input元素 
	$("tr:even")              选择所有的tr元素的偶数元素
	$("tr:odd")               选择所有的tr元素的奇数元素 
	$("td:eq(2)")             选择所有的td元素中序号为2的那个td元素 
	$("td:gt(4)")             选择td元素中序号大于4的所有td元素 
	$("td:lt(4)")             选择td元素中序号小于4的所有的td元素 
	$(":header") 
	$("div:animated") 
	
### 内容过滤选择器
	$("div:contains('John')") 选择所有div中含有John文本的元素 
	$("td:empty")             选择所有的为空（也不包括文本节点）的td元素的数组 
	$("div:has(p)")           选择所有含有p标签的div元素 
	$("td:parent")            选择所有的以td为父节点的元素数组 
	
### 可见性过滤选择器
	$("div:hidden")		选择所有的被hidden的div元素 
	$("div:visible")	选择所有的可视化的div元素 
	
### 属性过滤选择器
	$("div[id]")              		选择所有含有id属性的div元素 
	$("input[name='newsletter']")  选择所有的name属性等于'newsletter'的input元素 
	$("input[name!='newsletter']") 选择所有的name属性不等于'newsletter'的input元素 
	$("input[name^='news']")       选择所有的name属性以'news'开头的input元素 
	$("input[name$='news']")       选择所有的name属性以'news'结尾的input元素 
	$("input[name*='man']")        选择所有的name属性包含'news'的input元素 
	$("input[id][name$='man']")    可以使用多个属性进行联合选择，该选择器是得到所有的含有id属性并且那么属性以man结尾的元素 
 
### 子元素过滤选择器
	$("ul li:nth-child(2)")
	$("ul li:nth-child(odd)")
	$("ul li:nth-child(3n + 1)") 
	$("div span:first-child")        返回所有的div元素的第一个子节点的数组 
	$("div span:last-child")         返回所有的div元素的最后一个节点的数组 
	$("div button:only-child")       返回所有的div中只有唯一一个子节点的所有子节点的数组 
 
### 表单元素选择器
	$(":input")		选择所有的表单输入元素，包括input, textarea, select 和 button 
	$(":text")		选择所有的text input元素 
	$(":password")	选择所有的password input元素 
	$(":radio")		选择所有的radio input元素 
	$(":checkbox")	选择所有的checkbox input元素 
	$(":submit")	选择所有的submit input元素 
	$(":image")		选择所有的image input元素 
	$(":reset")		选择所有的reset input元素 
	$(":button")	选择所有的button input元素 
	$(":file")		选择所有的file input元素 
	$(":hidden")	选择所有类型为hidden的input元素或表单的隐藏域 
 
### 表单元素过滤选择器
	$(":enabled")             	选择所有的可操作的表单元素 
	$(":disabled")            	选择所有的不可操作的表单元素 
	$(":checked")            	选择所有的被checked的表单元素 
	$("select option:selected") 选择所有的select 的子元素中被selected的元素 
	
### 特例展示	
	选取一个 name 为”S_03_22″的input text框的上一个td的text值
	$(”input[@ name =S_03_22]“).parent().prev().text() 
	名字以”S_”开始，并且不是以”_R”结尾的
	$(”input[@ name ^='S_']“).not(”[@ name $='_R']“) 
	一个名为 radio_01的radio所选的值
	$(”input[@ name =radio_01][@checked]“).val(); 
	
 
