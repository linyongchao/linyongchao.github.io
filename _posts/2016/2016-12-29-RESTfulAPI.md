---
layout: post
title:  RESTful API 设计指南
date:   2016-12-29 17:59:59
categories: API
---

* content
{:toc}

本文来自：[阮一峰:RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)  
另外参考：[Restful API 的设计规范](http://www.tuicool.com/articles/bUZNZbZ)

## 协议

API与用户的通信协议，总是使用HTTPS协议。

## 域名

应该尽量将API部署在专用域名之下：

	https://api.example.com

如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下：

	https://example.org/api/

## 演进

### 版本

1. 将版本号放入URL：

		https://api.example.com/v1/

2. 将版本号放在HTTP头信息中：

		Accept Header： Accept: application/json+v1

3. 自定义 Header： 

		X-Api-Version: 1

### 失效

随着系统发展，总有一些API失效或者迁移  

* 对失效的API，返回 404 Not Found 或 410 Gone
* 对迁移的API，返回 301 重定向

## 路径

路径(URI)表示资源，一般对应服务器端领域模型中的实体类。  
URI表示资源有两种方式：资源集合、单个资源

### 规范

* 不用大写
* 用中杠 - 不用下杠 _
* 不要有动词，要用名词
* 名词使用复数形式
* 参数列表要encode
* 避免层级过深的URI

### 资源集合

	/zoos 		//所有动物园
	/zoos/1/animals //id为1的动物园中的所有动物

### 单个资源

	/zoos/1 	//id为1的动物园
	/zoos/1;2;3 	//id为1，2，3的动物园

### 举例

举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样：

	https://api.example.com/v1/zoos
	https://api.example.com/v1/animals
	https://api.example.com/v1/employees

过深的导航容易导致url膨胀，不易维护，应尽量使用查询参数代替路径中的实体导航，如：
	
	/zoos/1/areas/3/animals ==> /animals?zoo=1&area=3 

## Request

通过标准HTTP方法对资源进行CRUD。  
常用的HTTP方法如下：

	GET   （SELECT）：从服务器取出资源（一项或多项）
	POST  （CREATE）：在服务器新建一个资源
	PUT   （UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）
	PATCH （UPDATE）：在服务器更新资源（客户端提供改变的属性）
	DELETE（DELETE）：从服务器删除资源

	HEAD 	：获取资源的元数据（不常用）
	OPTIONS	：获取信息，关于资源的哪些属性是客户端可以改变的（不常用）

下面是一些例子：

	GET：查询
	GET /zoos		//查询所有动物园
	GET /zoos/1		//查询id为1的动物园
	GET /zoos/1/employees	//查询id为1的动物园的所有雇员
	
	POST：创建
	POST /zoos 		//创建一个动物园
	POST /zoos/1/employees  //为id为1的动物园创建雇员

	PUT：更新，客户端提供完整的更新后的资源
	PUT /zoos/1		//更新id为1的动物园信息
	
	PATCH：更新，客户端提供要更新的部分字段
	PATCH /zoos/1		//更新id为1的动物园信息

	DELETE：删除
	DELETE /zoos/1/animals  //删除id为1的动物园内的所有动物
	DELETE /zoos/1		//删除id为1的动物园

## Response

1. 符合HTTP规范

	| Method    | Response      |
	| ----------|:-------------:|
	| GET       | 单个对象、集合 	|
	| POST      | 新增成功的对象  	|
	| PUT/PATCH | 更新成功的对象  	|
	| DELETE 	| 空      		|

2. 不要包装

	Response 的 body 直接就是数据，不要做多余的包装。错误示例：

		{
    	"success":true,
    	"data":{"id":1,"name":"demo"},
		}

3. 时间用长整形

	返回一个毫秒数，客户端按需解析

4. 不传 NULL 字段
5. 分页

		{
    	"page":{"limit":10,"offset":0,"total":729},
    	"data":[{},{},{}...]
		}

## 错误处理

### 规范

1. 发生了错误不能给2xx响应，客户端可能会缓存成功的http请求
2. 正确设置HTTP状态码，不要自定义
3. Response body 提供 错误的代码（日志/问题追查）和 错误的描述文本（展示给用户）

### 解释

对上述规范的第三点稍微解释一下：  
API 可能抛出两类异常：业务异常和非业务异常。  

* 业务异常由自己的业务代码抛出，表示一个用例的前置条件不满足、业务规则冲突等，比如参数校验不通过、权限校验失败等。  
* 非业务类异常表示不在预期内的问题，通常由类库、框架抛出，或由于自己的代码逻辑错误导致，比如数据库连接失败、空指针异常、除0错误等。

业务类异常必须提供两种信息：

1. 异常的 HTTP 响应状态码
2. 异常的文本描述

实现时，在Controller层使用统一的异常拦截器：

1. 设置 HTTP 响应状态码  

	* 对业务类异常，用其指定的 HTTP code
	* 对非业务类异常，统一500

2. Response Body 的错误码：异常类名
3. Response Body 的错误描述
	
	* 对业务类异常，用其指定的文本描述
	* 对非业务类异常，线上可以统一文案如“服务器端错误，请稍后再试”；开发或测试环境中用异常的stacktrace，服务器端提供该行为的开关。

## 其他

* API的身份认证应该使用OAuth 2.0框架。  
* 服务器返回的数据格式，应该尽量使用JSON，避免使用XML。

- - -

## 附录

### 状态码

服务器向用户返回的状态码和提示信息，常见的如下：

	200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
	201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
	202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
	204 NO CONTENT - [DELETE]：用户删除数据成功。

	400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
	401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
	403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
	404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
	406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
	410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
	422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。

	500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。

### 安全性和幂等性

安全性 ：不会改变资源状态，可以理解为只读的  
幂等性 ：执行1次和执行N次，对资源状态改变的效果是等价的

| Method    | 安全性 | 幂等性 |
| ----------|:-----:|:-----:|
| GET       | √ 	| √ 	|
| POST      | ×  	| ×  	|
| PUT/PATCH | ×  	| √  	|
| DELETE 	| ×  	| √  	|

注意：安全性和幂等性均不保证反复请求能拿到相同的Response。  
以 DELETE 为例，第一次DELETE返回200表示删除成功，第二次返回404提示资源不存在，这是允许的。
