---
layout: post
title:  Tomcat 用户权限
date:   2017-08-18 10:54:54
categories: Tomcat
---

* content
{:toc}

Tomcat 权限大类上分为Manager和Admin两种

### Manager
1. manager-gui  
允许访问html接口(即URL路径为/manager/html/*)
2. manager-script  
允许访问纯文本接口(即URL路径为/manager/text/*)
3. manager-jmx  
允许访问JMX代理接口(即URL路径为/manager/jmxproxy/*)
4. manager-status  
允许访问Tomcat只读状态页面(即URL路径为/manager/status/*)  

ps. manager-gui、manager-script、manager-jmx均具备manager-status的权限  
即 manager-gui、manager-script、manager-jmx三种角色权限无需再额外添加manager-status权限，即可直接访问路径/manager/status/*

### Admin
1. admin-gui  
allows access to the HTML GUI
2. admin-script  
allows access to the text interface