---
layout: post
title:  大补课【2】switch
date:   2020-01-09 23:45:54
categories: Java
---

* content
{:toc}

### 前言

今天写一个 switch 判断，类型是枚举，结果报错，就补了下课  
参考[这里](https://blog.csdn.net/qq_37893505/article/details/91538833)和[这里](https://blog.csdn.net/zwt0909/article/details/52701283)

### 支持类型

|JDK版本|支持类型|变化|
|:---:|:---:|:---:|
|<1.5|byte、short、int、char|--|
|1.5|byte、short、int、char、enum|增加枚举|
|1.7|byte、short、int、char、enum、String|增加字符串|

* switch 支持的类型是逐渐增加的
* 也支持四种基本类型的封装类
* 封装类不能为 null，否则会报空指针异常

### 实现机制

* switch 底层是使用 int 类型来判断的
* int 类型是四个字节的整数型类型，只要字节小于或等于 4 的整数型类型都是可以转化成 int 类型，所以支持 byte（1字节），short（2字节）
* long（8字节）超出了 int 的范围，因而不支持
* 枚举和字符(串)也是转化为 int 类型间接实现的

写个例子，看看 switch 是如何依赖 int 类型支持字符串的，源码如下：

    public static void main(String[] args) {
        String abc = "通话";
        switch (abc) {
            case "重地":
                System.out.println("1=" + abc);
                break;
            case "123":
                System.out.println("3=" + abc);
                break;
            case "通话":
                System.out.println("2=" + abc);
                break;
            case "456":
                System.out.println("4=" + abc);
                break;
            case "abc":
                System.out.println("5=" + abc);
                break;
            default:
                System.out.println("----");
        }
    }

生成的 class 反编译如下：

    public static void main(String[] args) {
        String abc = "通话";
        byte var3 = -1;
        switch (abc.hashCode()) {
            case 48690:
                if (abc.equals("123")) {
                    var3 = 1;
                }
                break;
            case 51669:
                if (abc.equals("456")) {
                    var3 = 3;
                }
                break;
            case 96354:
                if (abc.equals("abc")) {
                    var3 = 4;
                }
                break;
            case 1179395:
                if (abc.equals("通话")) {
                    var3 = 2;
                } else if (abc.equals("重地")) {
                    var3 = 0;
                }
        }

        switch (var3) {
            case 0:
                System.out.println("1=" + abc);
                break;
            case 1:
                System.out.println("3=" + abc);
                break;
            case 2:
                System.out.println("2=" + abc);
                break;
            case 3:
                System.out.println("4=" + abc);
                break;
            case 4:
                System.out.println("5=" + abc);
                break;
            default:
                System.out.println("----");
        }
    }

其实比对源码与编译结果就应该懂了，不过还是总结一下：

* 执行两次 switch 语句，第一次将语句转换为 int，第二次才是真正的判断
* 第一次根据 case 条件的 hashCode() 值转换为 int，若 hashCode() 结果相同，则通过 equals 方法再判断一次
* 第一次的结果：如果符合第一个 case 条件，则返回 0；如果符合第二个 case 条件，则返回 1；……
* 第二次根据第一次的 int 结果进行判断

### 错误解决

今天写的 switch 报错如下：  
	
	An enum switch case label must be the unqualified name of an enumeration constant

即：case 后的枚举类型必须使用非限定名，那么去掉就解决问题了

