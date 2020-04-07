---
layout: post
title:  Spring AOP
date:   2020-04-07 18:59:54
categories: Java
---

* content
{:toc}

本文参考：[这里](https://blog.csdn.net/u010675669/article/details/91509247)、[这里](https://blog.csdn.net/weiweiai123456/article/details/54943724/)、[这里](https://zhuanlan.zhihu.com/p/100731870?utm_source=qq)、[这里](https://blog.csdn.net/u012843873/article/details/80540499)、[这里](https://blog.csdn.net/qq_15396517/article/details/84106997)

### 引包

下面这两个包必须同时引入，第二个包如果不引入，程序不会报错，但是不会进切面

	<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjrt -->
	<dependency>
	    <groupId>org.aspectj</groupId>
	    <artifactId>aspectjrt</artifactId>
	    <version>1.9.5</version>
	</dependency>
	<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjtools -->
	<dependency>
	    <groupId>org.aspectj</groupId>
	    <artifactId>aspectjtools</artifactId>
	    <version>1.9.5</version>
	</dependency>

### 定义注解

	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)
	public @interface TimeLog {
	    /**
	     * 方法描述
	     *
	     * @param
	     * @return
	     * @author lin
	     * @date 2020/4/7 12:03
	     **/
	    String description();
	}

### 定义切面

	@Aspect
	@Component
	public class TimeLogAspect {
	
	    @Pointcut("@annotation(my.learn.annotationtest.TimeLog)")
	    private void controllerAspect() {
	    }
	
	    @Around("controllerAspect()")
	    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
	        System.out.println("---around start---");
	        Object object = joinPoint.proceed();
	        System.out.println("---around end---");
	        return object;
	    }
	
	    @Before("controllerAspect()")
	    public void before() {
	        System.out.println("---before---");
	    }
	
	    @After("controllerAspect()")
	    public void after() {
	        System.out.println("---after---");
	    }
	
	    @AfterReturning(pointcut = "controllerAspect()", returning = "res")
	    public void afterReturning(JoinPoint joinPoint, Object res) throws Throwable {
	        System.out.println("---afterReturning start---");
	        //类名
	        String targetName = joinPoint.getTarget().getClass().getSimpleName();
	        System.out.println("类名:" + targetName);
	        MethodSignature ms = (MethodSignature) joinPoint.getSignature();
	        //方法名
	        String methodName = ms.getName();
	        System.out.println("方法名:" + methodName);
	        //入参key
	        String[] parameterNames = ms.getParameterNames();
	        //入参value
	        Object[] arguments = joinPoint.getArgs();
	        for (int i = 0; i < parameterNames.length; i++) {
	            System.out.println("参数" + i + ":" + parameterNames[i] + "=" + arguments[i]);
	        }
	        //方法的注解对象
	        Method method = ms.getMethod();
	        TimeLog timeLog = method.getAnnotation(TimeLog.class);
	        System.out.println("方法描述：" + timeLog.description());
	        System.out.println("方法名:" + method.getName());
	
	        //有类名、方法名、方法描述、参数，即可记录日志
	
	        System.out.println("---afterReturning end---");
	    }
	
	    @AfterThrowing("controllerAspect()")
	    public void afterThrowing() {
	        System.out.println("---afterThrowing---");
	    }
	
	}

### 使用注解

	@RestController
	@RequestMapping("/test")
	public class TestController {
	
	    @RequestMapping("/getNow")
	    @ResponseBody
	    @TimeLog(description = "获取当前时间")
	    public String getNow(@RequestParam(value = "userId") String userId) {
	        SimpleDateFormat sdf = new SimpleDateFormat(DateUtils.DEFAULT_PATTERN);
	        System.out.println("userId=" + userId);
	        return sdf.format(new Date());
	    }
	}

### 测试

打开URL ```http://localhost:12345/test/getNow?userId=6```
看日志输出如下：
```
---around start---
---before---
userId=6
---around end---
---after---
---afterReturning start---
类名:TestController
方法名:getNow
参数0:userId=6
方法描述：获取当前时间
方法名:getNow
---afterReturning end---
```

### 执行顺序

由上面的日志可以验证如下的切面执行顺序：
* 正常情况
![正常情况](https://img-blog.csdn.net/20160811192425854)
* 异常情况
![](https://img-blog.csdn.net/20160811192446479)

