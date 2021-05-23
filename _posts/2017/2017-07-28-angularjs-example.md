---
layout: post
title:  AngularJS应用实例
date:   2017-07-28 14:53:05
categories: WEB
---

* content
{:toc}

## 日期格式化

date_expression 即在$scope中设的date类型变量（注意，一定是date object才正确），就是要显示出来的日期  
| 是分割符号  
分割符号后面的第一个参数date是指明过滤器类型是过滤日期的  
第二个参数format是日期要格式化成什么样子，比如yyyy-MM-dd  
最后timezone一个是时区（可选参数），对于国际化的网站比较适用  

	{{ date_expression | date : format : timezone}}
	{{myDate | date:'medium'}}
	<h1 ng-bind="myDate | date:'yyyy-MM-dd'"></h1>
	var myJsDate=$filter('date')($scope.myDate,'yyyy-MM-dd');

## 数组数据重复

AngularJS 中repeat的数组数据不能出现重复项  

	<li ng-repeat="x in attr" ng-bind="x"></li>  

解决方法：  
在" x in attr "后面直接加上" track by $index "即可  

	<li ng-repeat="x in attr track by $index" ng-bind="x"></li>  