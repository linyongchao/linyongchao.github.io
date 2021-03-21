---
layout: post
title:  Spring AOP
date:   2021-03-21 22:39:05
categories: Java
---

* content
{:toc}

本文参考[这里](https://mp.weixin.qq.com/s/NXZp8a3n-ssnC6Y1Hy9lzw)  
去掉了趣味性的内容，只保留干货

### 什么是切面

我们知道，面向对象将程序抽象成多个层次的对象，每个对象负责不同的模块，这样的话各个对象分工明确，各司其职，也不互相藕合，有力地促进了工程开发与分工协作。  
但是新的问题来了，不同的模块（对象）间有时会出现公共的行为，这种公共的行为很难通过继承的方式来实现，如果用工具类的话也不利于维护，代码也显得异常繁琐。  
切面（AOP）的引入就是为了解决这类问题而生的，它要达到的效果是保证开发者在不修改源代码的前提下，为系统中不同的业务组件添加某些通用功能。

### 切面举例

![memory-layout](https://linyongchao.github.io/static/img/aip/1.webp)

比如上面这个例子，三个 service 对象执行过程中都存在安全、事务、日志、性能等相同行为，这些相同的行为显然应该在同一个地方管理。  
有人说我可以写一个统一的工具类，在这些对象的方法前/后都嵌入此工具类，那问题来了，这些行为都属于业务无关的，使用工具类嵌入的方式导致与业务代码紧藕合，很不合工程规范，代码可维护性极差！  
切面就是为了解决此类问题应运而生的，能做到相同功能的统一管理，对业务代码无侵入。

以性能为例，每个 service 都有不同的方法，我想统计每个方法的执行时间，如果不用切面你需要在每个方法的首尾计算下时间，然后相减：

![memory-layout](https://linyongchao.github.io/static/img/aip/2.webp)

如果我要统计每一个 service 中每个方法的执行时间可想而知不用切面的话就得在每个方法的首尾都加上类似上述的逻辑，显然这样的代码可维护性是非常差的。这还只是统计时间，如果此方法又要加上事务、风控等，是不是也得在方法首尾加上事务开始、回滚等代码，可想而知业务代码与非业务代码严重藕合，这样的实现方式对工程是一种灾难，是不能接受的！

### 与切面相关的几个定义

![memory-layout](https://linyongchao.github.io/static/img/aip/3.webp)

1. JoinPoint: 程序在执行流程中经过的一个个时间点，这个时间点可以是方法调用时，或者是执行方法中异常抛出时，也可以是属性被修改时等时机，在这些时间点上切面代码是可以（注意是可以但未必）被注入的

2. Pointcut: JoinPoints 只是切面代码可以被织入的地方，但并不想对所有的 JoinPoint 进行织入，这就需要某些条件来筛选出那些需要被织入的 JoinPoint，Pointcut 就是通过一组规则(使用 AspectJ pointcut expression language 来描述) 声明式表达式来定位到匹配的 joinpoint，这个表达式支持的类型比较全面，可以用正则，注解等来指定满足条件的 joinpoint , 比如类名后加 .*(..) 这样的正则表达式就代表这个类里面的所有方法都会被织入，使用 @annotation 的方式也可以指定对标有这类注解的方法织入代码

3. Advice:  代码织入（也叫增强），Pointcut 通过其规则指定了哪些 joinpoint 可以被织入，而 Advice 则指定了这些 joinpoint 被织入（或者增强）的具体时机与逻辑，是切面代码真正被执行的地方，主要有五个织入时机

	* Before Advice: 在 JoinPoints 执行前织入
	* After Advice: 在 JoinPoints 执行后织入（不管是否抛出异常都会织入）
	* After returning advice: 在 JoinPoints 执行正常退出后织入（抛出异常则不会被织入）
	* After throwing advice: 方法执行过程中抛出异常后织入
	* Around Advice: 这是所有 Advice 中最强大的，它在 JoinPoints 前后都可织入切面代码，也可以选择是否执行原有正常的逻辑，如果不执行原有流程，它甚至可以用自己的返回值代替原有的返回值，甚至抛出异常。在这些 advice 里我们就可以写入切面代码了

综上所述，切面（Aspect）我们可以认为就是 pointcut 和 advice，pointcut 指定了哪些 joinpoint 可以被织入，而 advice 则指定了在这些 joinpoint 上的代码织入时机与逻辑

画外音：织入（weaving），将切面作用于委托类对象以创建 adviced object 的过程（即代理，下文会提）

### 实例1

比如在餐馆里点菜，菜单有 10 个菜，这 10 个菜就是 JoinPoint  
但我只点了带有萝卜名字的菜，那么带有萝卜名字这个条件就是针对 JoinPoint（10 个菜）的筛选条件，即 pointcut，最终只有胡萝卜，白萝卜这两个 JoinPoint 满足条件  
然后我们就可以在吃胡萝卜前洗手（before advice），或吃胡萝卜后买单（after advice），也可以统计吃胡萝卜的时间（around advice），这些洗手，买单，统计时间的动作都是与吃萝卜这个业务动作解藕的,都是统一写在 advice 的逻辑里

代码如下：

	public interface TestService {
	   // 吃萝卜
	   void eatCarrot();
	
	   // 吃蘑菇
	   void eatMushroom();
	
	   // 吃白菜
	   void eatCabbage();
	}
	
	@Component
	public class TestServiceImpl implements TestService {
	   @Override
	   public void eatCarrot() {
	       System.out.println("吃萝卜");
	   }
	
	   @Override
	   public void eatMushroom() {
	       System.out.println("吃蘑菇");
	   }
	
	   @Override
	   public void eatCabbage() {
	       System.out.println("吃白菜");
	   }
	}

假设有以上 TestService, 实现了吃萝卜、吃蘑菇、吃白菜三个方法，这三个方法都用切面织入，所以它们都是 joinpoints，但现在我只想对吃萝卜这个 joinpoints 前后织入 advice，该怎么办呢？  
首先当然要声明 pointcut 表达式，这个表达式表明只想织入吃萝卜这个 joinpoint，指明了之后再让 advice 应用于此 pointcut，比如我想在吃萝卜前洗手、吃萝卜后买单，可以写出如下切面逻辑

	@Aspect
	@Component
	public class TestAdvice {
	   // 1. 定义 PointCut
	   @Pointcut("execution(* com.example.demo.api.TestServiceImpl.eatCarrot())")
	   private void eatCarrot(){}
	
	   // 2. 定义应用于 JoinPoint 中所有满足 PointCut 条件的 advice, 这里我们使用 around advice，在其中织入增强逻辑
	   @Around("eatCarrot()")
	   public void handlerRpcResult(ProceedingJoinPoint point) throws Throwable {
	       System.out.println("吃萝卜前洗手");
	       //  原来的 TestServiceImpl.eatCarrot 逻辑，可视情况决定是否执行
	       point.proceed();
	       System.out.println("吃萝后买单");
	   }
	}
	
可以看到通过 AOP 我们巧妙地在方法执行前后执行插入相关的逻辑，对原有执行逻辑无任何侵入！

### 实例2

某个对外提供的服务可能有多个方法，一般我们为了不给调用方埋坑，会在每个方法里把所有异常都 catch 住，只返回一个 result，调用方会根据这个 result 里的 success 判断此次调用是否成功，举个例子

	public class ServiceResultTO<T> extends Serializable {
	   private static final long serialVersionUID = xxx;
	   private Boolean success;
	   private String message;
	   private T data;
	}
	
	public interface TestService {
	   ServiceResultTO<Boolean> test();
	}
	
	public class TestServiceImpl implements TestService {
	   @Override
	   public ServiceResultTO<Boolean> test() {
	       try {
	           // 此处写服务里的执行逻辑
	           return ServiceResultTO.buildSuccess(Boolean.TRUE);
	       } catch(Exception e) {
	         return ServiceResultTO.buildFailed(Boolean.FALSE, "执行失败");            
	       }
	   }
	}
	
以上这样的服务（TestService），它有一个 test 方法，我们在此方法执行逻辑外包了一层「try... catch...」  
如果只有一个 test 方法，这样做当然没问题，但问题是在工程里我们一般要提供几十上百个 service，每个 service 有几十个像 test 这样的方法，如果每个方法都要在执行的时候包一层 「try ...catch...」，虽然可行，但代码会比较丑陋，可读性也比较差

改造步骤如下：

首先我们先定义一个如下注解

	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)
	public @interface GlobalErrorCatch {
	
	}

然后将所有 service 中方法里的 「try... catch...」移除掉，在方法签名上加上上述我们定义好的注解

	public class TestServiceImpl implements TestService {
	   @Override
	   @GlobalErrorCatch
	   public ServiceResultTO<Boolean> test() {
	        // 此处写服务里的执行逻辑
	        boolean result = xxx;
	        return ServiceResultTO.buildSuccess(result);
	   }
	}

然后再指定注解形式的 pointcuts 及 around advice

	@Aspect
	@Component
	public class TestAdvice {
	   // 1. 定义所有带有 GlobalErrorCatch 的注解的方法为 Pointcut
	   @Pointcut("@annotation(com.example.demo.annotation.GlobalErrorCatch)")
	   private void globalCatch(){}
	   // 2. 将 around advice 作用于 globalCatch(){} 此 PointCut 
	   @Around("globalCatch()")
	   public Object handlerGlobalResult(ProceedingJoinPoint point) throws Throwable {
	       try {
	           return point.proceed();
	       } catch (Exception e) {
	           System.out.println("执行错误" + e);
	           return ServiceResultTO.buildFailed("系统错误");
	       }
	   }
	
	}
	
通过这样的方式，所有标记着 GlobalErrorCatch 注解的方法都会统一在 handlerGlobalResult 方法里执行，我们就可以在这个方法里统一 catch 住异常，所有 service 方法中又长又臭的 「try...catch...」全部干掉，真香！

### AOP实现原理

首先打印 TestServiceImp 这个 bean 所属的类

	@Component
	public class TestServiceImpl implements TestService {
	   @Override
	   public void eatCarrot() {
	       System.out.println("吃萝卜");
	   }
	}
	
	@Aspect
	@Component
	public class TestAdvice {
	   // 1. 定义 PointCut
	   @Pointcut("execution(* com.example.demo.api.TestServiceImpl.eatCarrot())")
	   private void eatCarrot(){}
	
	   // 2. 定义应用于 PointCut 的 advice, 这里我们使用 around advice
	   @Around("eatCarrot()")
	   public void handlerRpcResult(ProceedingJoinPoint point) throws Throwable {
	        // 省略相关逻辑
	   }
	}
	
	@SpringBootApplication
	@EnableAspectJAutoProxy
	public class DemoApplication {
	   public static void main(String[] args) {
	       ConfigurableApplicationContext context = SpringApplication.run(DemoApplication.class, args);
	       TestService testService = context.getBean(TestService.class);
	       System.out.println("testService = " + testService.getClass());
	   }
	}

![memory-layout](https://linyongchao.github.io/static/img/aip/4.webp)

打印后发现这个 bean 的 class 居然不是 TestServiceImpl！而是

	com.example.demo.impl.TestServiceImplEnhancerBySpringCGLIB$$705c68c7

注意到类名中有一个 EnhancerBySpringCGLIB ，注意 CGLiB，这个类就是通过它生成的动态代理

### 代理

代理在生活中随处可见，比如说我要买房，我一般不会直接和卖家对接，一般会和中介打交道，中介就是代理，卖家就是目标对象，我就是调用者，代理不仅实现了目标对象的行为（帮目标对象卖房），还可以添加上自己的动作（收保证金，签合同等）：

![memory-layout](https://linyongchao.github.io/static/img/aip/5.webp)

用 UML 图来表示就是下面这样：

![memory-layout](https://linyongchao.github.io/static/img/aip/6.webp)

Client 是直接和 Proxy 打交道的，Proxy 是 Client 要真正调用的 RealSubject 的代理，它确实执行了 RealSubject 的 request 方法，不过在这个执行前后 Proxy 也加上了额外的 PreRequest()，afterRequest() 方法，注意 Proxy 和 RealSubject 都实现了 Subject 这个接口，这样在 Client 看起来调用谁是没有什么分别的（面向接口编程，对调用方无感，因为实现的接口方法是一样的），Proxy 通过其属性持有真正要代理的目标对象（RealSubject）以达到既能调用目标对象的方法也能在方法前后注入其它逻辑的目的

代理主要分为两种类型：静态代理和动态代理，动态代理又有 JDK 代理和 CGLib 代理两种

要理解静态和动态这两个含义，我们首先需要理解一下 Java 程序的运行机制

![memory-layout](https://linyongchao.github.io/static/img/aip/7.webp)

首先 Java 源代码经过编译生成字节码，然后再由 JVM 经过类加载，连接，初始化成 Java 类型，可以看到字节码是关键，静态和动态的区别就在于字节码生成的时机  

静态代理：由程序员创建代理类或特定工具自动生成源代码再对其编译。在编译时已经将接口，被代理类（委托类），代理类等确定下来，在程序运行前代理类的.class文件就已经存在了  
动态代理：在程序运行后通过反射创建生成字节码再由 JVM 加载而成

按这张 UML 类库依葫芦画瓢，傻瓜也会

![memory-layout](https://linyongchao.github.io/static/img/aip/6.webp)


	public interface Subject {
	   public void request();
	}
	
	public class RealSubject implements Subject {
	   @Override
	   public void request() {
	       // 卖房
	       System.out.println("卖房");
	   }
	}
	
	public class Proxy implements Subject {
	
	   private RealSubject realSubject;
	
	   public Proxy(RealSubject subject) {
	       this.realSubject = subject;
	   }
	
	
	   @Override
	   public void request() {
	    // 执行代理逻辑
	       System.out.println("卖房前");
	
	       // 执行目标对象方法
	       realSubject.request();
	
	       // 执行代理逻辑
	       System.out.println("卖房后");
	   }
	
	   public static void main(String[] args) {
	       // 被代理对象
	       RealSubject subject = new RealSubject();
	
	       // 代理
	       Proxy proxy = new Proxy(subject);
	
	       // 代理请求
	       proxy.request();
	   }
	}

静态代理的两大劣势

1. 代理类只代理一个委托类（其实可以代理多个，但不符合单一职责原则），也就意味着如果要代理多个委托类，就要写多个代理（别忘了静态代理在编译前必须确定）
2. 第一点还不是致命的，再考虑这样一种场景：如果每个委托类的每个方法都要被织入同样的逻辑，比如说我要计算前文提到的每个委托类每个方法的耗时，就要在方法开始前，开始后分别织入计算时间的代码，那就算用代理类，它的方法也有无数这种重复的计算时间的代码

### 动态代理

静态代理的这些劣势主要是因为在编译前这些代理类是确定的，如果这些代理类是动态生成的呢，就可以省略一大堆代理的代码

动态代理分为 JDK 提供的动态代理和 Spring AOP 用到的 CGLib 生成的代理  

我们先看下 JDK 提供的动态代理该怎么写，这是代码

	// 委托类
	public class RealSubject implements Subject {
	   @Override
	   public void request() {
	       // 卖房
	       System.out.println("卖房");
	   }
	}
	
	
	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	import java.lang.reflect.Proxy;
	
	public class ProxyFactory {
	
	    // 维护一个目标对象
	    private Object target;
	
	    public ProxyFactory(Object target) {
	        this.target = target;
	    }
	
	    // 为目标对象生成代理对象
	    public Object getProxyInstance() {
	        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),
	            (Object proxy, Method method, Object[] args) -> {
	                System.out.println("计算开始时间");
	                // 执行目标对象方法
	                method.invoke(target, args);
	                System.out.println("计算结束时间");
	                return null;
	            });
	    }
	
	    public static void main(String[] args) {
	        RealSubject realSubject = new RealSubject();
	        System.out.println(realSubject.getClass());
	        Subject subject = (Subject) new ProxyFactory(realSubject).getProxyInstance();
	        System.out.println(subject.getClass());
	        subject.request();
	    }
	}

打印结果如下:

	class com.service.impl.RealSubject
	class com.sun.proxy.$Proxy0
	计算开始时间
	卖房
	计算结束时间

我们注意到代理类的 class 为 ```com.sun.proxy.$Proxy0```，它是如何生成的呢？  
注意到 Proxy 是在 java.lang.reflect 反射包下的，注意看看 Proxy 的 newProxyInstance 签名

	public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h);

1. loader: 代理类的ClassLoader，最终读取动态生成的字节码，并转成 java.lang.Class 类的一个实例（即类），通过此实例的 newInstance() 方法就可以创建出代理的对象
2. interfaces: 委托类实现的接口，JDK 动态代理要实现所有的委托类的接口
3. InvocationHandler: 委托对象所有接口方法调用都会转发到InvocationHandler.invoke()，在 invoke() 方法里我们可以加入任何需要增强的逻辑 主要是根据委托类的接口等通过反射生成的

这样的实现有啥好处呢？

由于动态代理是程序运行后才生成的，哪个委托类需要被代理到，只要生成动态代理即可，避免了静态代理那样的硬编码，另外所有委托类实现接口的方法都会在 Proxy 的 InvocationHandler.invoke() 中执行，这样如果要统计所有方法执行时间这样相同的逻辑，可以统一在 InvocationHandler 里写， 也就避免了静态代理那样需要在所有的方法中插入同样代码的问题，代码的可维护性极大的提高了。

说得这么厉害，那么 Spring AOP 的实现为啥却不用它呢？

JDK 动态代理虽好，但也有弱点，我们注意到 newProxyInstance 的方法签名

	public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h);

注意第二个参数 Interfaces 是委托类的接口，是必传的， JDK 动态代理是通过与委托类实现同样的接口，然后在实现的接口方法里进行增强来实现的，这就意味着如果要用 JDK 代理，委托类必须实现接口，这样的实现方式看起来有点蠢，更好的方式是什么呢，直接继承自委托类不就行了，这样委托类的逻辑不需要做任何改动，CGlib 就是这么做的

### CGLib 动态代理

JDK 动态代理使用 Proxy 来创建代理类，增强逻辑写在 InvocationHandler.invoke() 里，CGlib 动态代理也提供了类似的  Enhance 类，增强逻辑写在 MethodInterceptor.intercept() 中，也就是说所有委托类的非 final 方法都会被方法拦截器拦截，在说它的原理之前首先来看看它怎么用的

	public class MyMethodInterceptor implements MethodInterceptor {
	   @Override
	   public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
	       System.out.println("目标类增强前！！！");
	       //注意这里的方法调用，不是用反射哦！！！
	       Object object = proxy.invokeSuper(obj, args);
	       System.out.println("目标类增强后！！！");
	       return object;
	   }
	}
	
	public class CGlibProxy {
	   public static void main(String[] args) {
	       //创建Enhancer对象，类似于JDK动态代理的Proxy类，下一步就是设置几个参数
	       Enhancer enhancer = new Enhancer();
	       //设置目标类的字节码文件
	       enhancer.setSuperclass(RealSubject.class);
	       //设置回调函数
	       enhancer.setCallback(new MyMethodInterceptor());
	
	       //这里的creat方法就是正式创建代理类
	       RealSubject proxyDog = (RealSubject) enhancer.create();
	       //调用代理类的eat方法
	       proxyDog.request();
	   }
	}
	
打印如下

	代理类:class com.example.demo.proxy.staticproxy.RealSubject$$EnhancerByCGLIB$$889898c5
	目标类增强前！！！
	卖房
	目标类增强后！！！
	
可以看到主要就是利用 Enhancer 这个类来设置委托类与方法拦截器，这样委托类的所有非 final 方法就能被方法拦截器拦截，从而在拦截器里实现增强

CGLib 的底层实现原理

之前也说了它是通过继承自委托类，重写委托类的非 final 方法（final 方法不能重载），并在方法里调用委托类的方法来实现代码增强的，它的实现大概是这样

	public class RealSubject {
	   @Override
	   public void request() {
	       // 卖房
	       System.out.println("卖房");
	   }
	}
	
	/** 生成的动态代理类（简化版）**/
	public class RealSubject$$EnhancerByCGLIB$$889898c5 extends RealSubject {
	   @Override
	   public void request() {
	       System.out.println("增强前");
	       super.request();
	       System.out.println("增强后");
	   }
	}

可以看到它并不要求委托类实现任何接口，而且 CGLIB 是高效的代码生成包，底层依靠 ASM（开源的 java 字节码编辑类库）操作字节码实现的，性能比 JDK 强，所以 Spring AOP 最终使用了 CGlib 来生成动态代理

CGlib 动态代理使用上有啥限制吗？

第一点之前已经已经说了，只能代理委托类中任意的非 final 的方法，另外它是通过继承自委托类来生成代理的，所以如果委托类是 final 的，就无法被代理了（final 类不能被继承）

JDK 动态代理的拦截对象是通过反射的机制来调用被拦截方法的，CGlib 呢，它通过什么机制来提升了方法的调用效率？  
由于反射的效率比较低，所以 CGlib 采用了FastClass 的机制来实现对被拦截方法的调用。FastClass 机制就是对一个类的方法建立索引，通过索引来直接调用相应的方法，建议参考下```https://www.cnblogs.com/cruze/p/3865180.html```这个链接好好学学


还有一个问题，我们通过打印类名的方式知道了 cglib 生成了 RealSubjectEnhancerByCGLIB$$889898c5 这样的动态代理，那么有反编译过它的 class 文件来了解 cglib 代理类的生成规则吗？
也在参考链接里

### 后记

AOP 是 Spring 一个非常重要的特性，通过切面编程有效地实现了不同模块相同行为的统一管理，也与业务逻辑实现了有效解藕，善用 AOP 有时候能起到出奇制胜的效果  
举一个例子，我们业务中有这样的一个需求，需要在不同模块中一些核心逻辑执行前过一遍风控，风控通过了，这些核心逻辑才能执行，怎么实现呢？  
你当然可以统一封装一个风控工具类，然后在这些核心逻辑执行前插入风控工具类的代码，但这样的话核心逻辑与非核心逻辑（风控，事务等）就藕合在一起了，更好的方式显然应该用 AOP，使用文中所述的注解 + AOP 的方式，将这些非核心逻辑解藕到切面中执行，让代码的可维护性大大提高了。

篇幅所限，文中没有分析 JDK 和 CGlib 的动态代理生成的实现，不过建议大家有余力的话还是可以看看，尤其是文末的参考链接，生成动态代理主要用到了反射的特性，不过我们知道反射存在一定的性能问题，为了提升性能，底层用了一些比如缓存字节码，FastClass 之类的技术来提升性能，通读源码之后的，对反射的理解也会大大加深。

参考资料

Spring AOP 是怎么运行的？彻底搞定这道面试必考题 https://cloud.tencent.com/developer/article/1584491
