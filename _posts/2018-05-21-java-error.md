---
layout: post
title:  Java 错误
date:   2018-05-21 22:08:54
categories: Java
---

* content
{:toc}

### Illegal key size

异常：java.security.InvalidKeyException: Illegal key size  
解决：密钥长度修改为128位或者使用JCE无限制权限策略文件   
原因：如果密钥大于128位，会抛出```java.security.InvalidKeyException: Illegal key size```异常。因为密钥长度是受限制的，java运行时环境读到的是受限的policy文件。文件位于```${java_home}/jre/lib/security```，这种限制是因为美国对软件出口的控制。   
替换：去官方下载JCE无限制权限策略文件，下载地址[JDK 5](http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-java-plat-419418.html#jce_policy-1.5.0-oth-JPR)、[JDK 6](http://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html)、[JDK 7](http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html)、[JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)。  
下载后解压，可以看到```local_policy.jar```和```US_export_policy.jar```以及```readme.txt```  
如果安装了JRE，将两个jar文件放到```%JRE_HOME%\lib\security```目录下覆盖原来的文件  
如果安装了JDK，还要将两个jar文件也放到```%JDK_HOME%\jre\lib\security```目录下覆盖原来文件。  
参考：[这里](https://www.cnblogs.com/lilinzhiyu/p/8024100.html)

### Given final block not properly padded

异常：javax.crypto.BadPaddingException: Given final block not properly padded  
错误代码：

	private Key initKeyForAES(String key) throws NoSuchAlgorithmException {
	    if (null == key || key.length() == 0) {
	        throw new NullPointerException("key not is null");
	    }
	    SecretKeySpec key2 = null;
	    try {
	        KeyGenerator kgen = KeyGenerator.getInstance("AES");
	        kgen.init(128, new SecureRandom(key.getBytes()));
	        SecretKey secretKey = kgen.generateKey();
	        byte[] enCodeFormat = secretKey.getEncoded();
	        key2 = new SecretKeySpec(enCodeFormat, "AES");
	    } catch (NoSuchAlgorithmException ex) {
	        throw new NoSuchAlgorithmException();
	    }
	    return key2;
	
	}

问题修复：

	private Key initKeyForAES(String key) throws NoSuchAlgorithmException {
	    if (null == key || key.length() == 0) {
	        throw new NullPointerException("key not is null");
	    }
	    SecretKeySpec key2 = null;
	    SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
	    random.setSeed(key.getBytes());
	    try {
	        KeyGenerator kgen = KeyGenerator.getInstance("AES");
	        kgen.init(128, random);
	        SecretKey secretKey = kgen.generateKey();
	        byte[] enCodeFormat = secretKey.getEncoded();
	        key2 = new SecretKeySpec(enCodeFormat, "AES");
	    } catch (NoSuchAlgorithmException ex) {
	        throw new NoSuchAlgorithmException();
	    }
	    return key2;
	
	}
   
解决：
即，将
	
	new SecureRandom(key.getBytes())
	
替换为

	SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
	random.setSeed(key.getBytes());
  
参考：[这里](https://www.cnblogs.com/zempty/p/4318902.html)
