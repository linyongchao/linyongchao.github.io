---
layout: post
title:  Redis pipeline
date:   2020-04-15 16:59:54
categories: Redis
---

* content
{:toc}

本文参考：[这里](https://blog.csdn.net/u011489043/article/details/78769428)

### 简介

Redis 使用的是客户端-服务器（CS）模型和请求/响应协议的 TCP 服务器。这意味着通常情况下一个请求会遵循以下步骤：

1. 客户端向服务端发送一个查询请求，并监听 Socket 返回，通常是以阻塞模式，等待服务端响应。
2. 服务端处理命令，并将结果返回给客户端。

Redis 客户端与 Redis 服务器之间使用 TCP 协议进行连接，一个客户端可以通过一个 Socket 连接发起多个请求命令。每个请求命令发出后客户端通常会阻塞并等待服务器处理，服务器处理完请求命令后会将结果通过响应报文返回给客户端，因此当执行多条命令的时候都需要等待上一条命令执行完毕才能执行。

其执行过程如下图所示：

![](https://linyongchao.github.io/static/img/redis/pipeline1.jpg)

由于通信会有网络延迟，假如客户端和服务端之间的包传输时间需要0.125秒。那么上面的三个命令6个报文至少需要0.75秒才能完成。这样即使服务端每秒能处理100个命令，而我们的客户端也只能一秒钟发出四个命令。这显然没有充分利用服务端的处理能力。

而管道（pipeline）可以一次性发送多条命令并在执行完后一次性将结果返回，pipeline 通过减少客户端与服务端的通信次数来实现降低往返延时时间的目的，而且 pipeline 实现的原理是队列，而队列的原理是先进先出，这样就保证数据的顺序性。pipeline 的默认同步个数为53个，也就是说 arges 中累加到53条数据时会把数据提交。下图所示客户端可以将三个命令放到一个 tcp 报文一起发送，服务端则可以将三条命令的处理结果放到一个 tcp 报文返回：

![](https://linyongchao.github.io/static/img/redis/pipeline2.jpg)

需要注意的是：用 pipeline 方式打包命令发送，服务端必须在处理完所有命令前先缓存起所有命令的处理结果。打包的命令越多，缓存消耗内存也越多，所以并不是打包的命令越多越好。具体多少合适需要根据具体情况测试。

### 代码测试

	import redis.clients.jedis.Jedis;
	import redis.clients.jedis.Pipeline;
	
	public class RedisUtils {
	
	    public static void main(String[] args) {
	        int num = 1000000;
	        Jedis jedis = new Jedis();
	
	        long start = System.currentTimeMillis();
	        for (int i = 0; i < num; i++) {
	            jedis.lpush("seq1", "abc" + i);
	        }
	        long middle = System.currentTimeMillis();
	        System.out.println("第一队列用时：" + (middle - start) + "毫秒");
	
	        Pipeline pipeline = jedis.pipelined();
	        for (int i = 0; i < num; i++) {
	            pipeline.lpush("seq2", "abc" + i);
	        }
	        pipeline.sync();
	        long end = System.currentTimeMillis();
	        System.out.println("第二队列用时：" + (end - middle) + "毫秒");
	    }
	}

代码运行结果为：

	第一队列用时：18218毫秒
	第二队列用时：6146毫秒

由于使用的是本地的的 Redis 服务器，pipeline 的优势不明显，但即使如此，也节省了三分之二的时间

### 适用场景

* 有些系统可能对可靠性要求很高，每次操作都需要立刻知道这次操作是否成功，是否数据已经写进 Redis 了，那这种场景就不适合。  
* 有的系统，可能是批量的将数据写入 Redis，允许一定比例的写入失败，那么这种场景就可以使用了，比如10000条数据同时进入 Redis，可能失败了2条无所谓，后期有补偿机制就可以了。  
比如短信群发这种场景，如果一下群发10000条，按照第一种模式去实现，要很久才能给客户端响应，如果客户端请求设置了超时时间5秒，那肯定就抛出异常了，而且本身群发短信要求实时性也没那么高，这时候用 pipeline 最好了。
