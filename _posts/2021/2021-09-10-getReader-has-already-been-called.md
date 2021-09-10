---
layout: post
title:  getReader() has already been called
date:   2021-09-10 11:39:05
categories: Java
---

* content
{:toc}

解决方案参考[这里](https://www.cnblogs.com/alter888/p/8919865.html)

### 背景

项目中遇到一个需求：对 WEB 页面传递的参数进行校验  
即：WEB 页面在传递参数的同时，会通过 Authorization 传递校验的 MD5；服务端在拿到参数之后也计算 MD5，如果和 WEB 页面传递的不同，则说明被篡改，校验不通过

这种需求肯定要用 Interceptor 或者 Filter 的，结果在增加校验之后，之前就存在的 Filter 报错了（完整错误见附录）：

	java.lang.IllegalStateException: getReader() has already been called for this request
	
原来，新增的校验是通过```request.getReader()```方法获取的参数，之前就存在的 Filter 也使用该方法获取参数  
但是，```request.getReader()```和```request.getInputStream()```只能调用一次，第二次调用即报上述错误

解决办法：先将 RequestBody 保存为一个 byte 数组，然后通过 Servlet 自带的 ```HttpServletRequestWrapper``` 类覆盖 ```getReader()```和```getInputStream()``` 方法，使流从保存的 byte 数组中读取，最后在 Filter 中将 ServletRequest 替换为 ```HttpServletRequestWrapper```

### 代码

1. 重写 HttpServletRequestWrapper

		import java.io.BufferedReader;
		import java.io.ByteArrayInputStream;
		import java.io.IOException;
		import java.io.StringWriter;
		import javax.servlet.ReadListener;
		import javax.servlet.ServletInputStream;
		import javax.servlet.http.HttpServletRequest;
		import javax.servlet.http.HttpServletRequestWrapper;
		
		public class ParamHttpServletRequestWrapper extends HttpServletRequestWrapper {
		
		    private byte[] body;
		
		    public ParamHttpServletRequestWrapper(HttpServletRequest request) throws IOException {
		        super(request);
		
		        BufferedReader reader = request.getReader();
		        try (StringWriter writer = new StringWriter()) {
		            int read;
		            char[] buf = new char[1024 * 8];
		            while ((read = reader.read(buf)) != -1) {
		                writer.write(buf, 0, read);
		            }
		            this.body = writer.getBuffer().toString().getBytes();
		        }
		    }
		
		    public byte[] getBody() {
		        return body;
		    }
		
		    @Override
		    public javax.servlet.ServletInputStream getInputStream() throws IOException {
		        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(body);
		        return new ServletInputStream() {
		
		            @Override
		            public int read() throws IOException {
		                return byteArrayInputStream.read();
		            }
		
		            @Override
		            public void setReadListener(ReadListener listener) {
		                this.setReadListener(listener);
		            }
		
		            @Override
		            public boolean isReady() {
		                return false;
		            }
		
		            @Override
		            public boolean isFinished() {
		                return false;
		            }
		        };
		    }
		}

2. 实现一个全局 Filter 将 HttpServletRequest 转换为 ParamHttpServletRequestWrapper
		
		import java.io.IOException;
		import javax.servlet.Filter;
		import javax.servlet.FilterChain;
		import javax.servlet.FilterConfig;
		import javax.servlet.ServletException;
		import javax.servlet.ServletRequest;
		import javax.servlet.ServletResponse;
		import javax.servlet.http.HttpServletRequest;
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;
		import org.springframework.stereotype.Component;
		
		@Component
		public class ParamFilter implements Filter {
		
		    private Logger logger = LoggerFactory.getLogger(ParamFilter.class);
		
		    @Override
		    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		        logger.info("=== doFilter ===");
		        if (request instanceof HttpServletRequest) {
		            ServletRequest requestWrapper = new ParamHttpServletRequestWrapper((HttpServletRequest) request);
		            chain.doFilter(requestWrapper, response);
		        } else {
		            chain.doFilter(request, response);
		        }
		    }
		
		    @Override
		    public void init(FilterConfig filterConfig) throws ServletException {
		        logger.info("ParamFilter init ……");
		    }
		
		    @Override
		    public void destroy() {
		        logger.info("ParamFilter destroy ……");
		    }
		}

3. 实现一个 Interceptor 完成校验功能
		
		import com.alibaba.fastjson.JSONObject;
		import java.io.UnsupportedEncodingException;
		import java.net.URLDecoder;
		import java.util.HashMap;
		import java.util.Map;
		import java.util.TreeMap;
		import javax.servlet.http.HttpServletRequest;
		import javax.servlet.http.HttpServletResponse;
		import org.apache.commons.lang3.StringUtils;
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;
		import org.springframework.stereotype.Component;
		import org.springframework.web.servlet.HandlerInterceptor;
		
		@Component
		public class ParamCheckInterceptor implements HandlerInterceptor {
		
		    private Logger logger = LoggerFactory.getLogger(ParamCheckInterceptor.class);
		
		    @Override
		    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		        logger.info("进入ParamCheckInterceptor……");
		        String authorization = request.getHeader("Authorization");
		        String md5 = null; // 从 authorization 中获取
		        String param = getParam(request);
		        String checkMd5 = null;// 根据 param 获取 md5
		        return md5.equals(checkMd5);
		    }
		
		    private String getParam(HttpServletRequest request) throws UnsupportedEncodingException {
		        Map<String, String> result = new HashMap<>();
		        if ("POST".equals(request.getMethod())) {
		            if (request instanceof ParamHttpServletRequestWrapper) {
		                ParamHttpServletRequestWrapper requestWrapper = (ParamHttpServletRequestWrapper) request;
		                String param = new String(requestWrapper.getBody());
		                if (StringUtils.isNotEmpty(param)) {
		                    result = JSONObject.parseObject(param, Map.class);
		                }
		            }
		        } else if ("GET".equals(request.getMethod())) {
		            String param = URLDecoder.decode(request.getQueryString(), "utf-8");
		            String[] params = param.split("&");
		            for (String s : params) {
		                Integer index = s.indexOf('=');
		                result.put(s.substring(0, index), s.substring(index + 1));
		            }
		        }
		        TreeMap<String, Object> treeMap = new TreeMap(result);
		        StringBuilder sb = new StringBuilder();
		        for (Map.Entry<String, Object> entrySet : treeMap.entrySet()) {
		            sb.append(entrySet.getKey()).append("=").append(entrySet.getValue().toString()).append("&");
		        }
		        if (sb.length() > 1) {
		            sb.deleteCharAt(sb.length() - 1);
		        }
		        return sb.toString();
		    }
		}

4. 配置 Interceptor 的使用范围为全局
		
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.stereotype.Component;
		import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
		import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
		
		@Component
		public class ParamCheckConfiguration implements WebMvcConfigurer {
		
		    @Autowired
		    private ParamCheckInterceptor paramCheckInterceptor;
		
		    @Override
		    public void addInterceptors(InterceptorRegistry registry) {
		        registry.addInterceptor(paramCheckInterceptor).addPathPatterns("/**");
		    }
		}

	
### 附录

完整错误：

	2021-09-02 14:57:27,262 ERROR [http-nio-9681-exec-4] [mdm-DirectJDKLog.java:175]: Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.IllegalStateException: getReader() has already been called for this request] with root cause
	java.lang.IllegalStateException: getReader() has already been called for this request
		at org.apache.catalina.connector.Request.getInputStream(Request.java:1049)
		at org.apache.catalina.connector.RequestFacade.getInputStream(RequestFacade.java:365)
		at org.springframework.http.server.ServletServerHttpRequest.getBody(ServletServerHttpRequest.java:211)
		at org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodArgumentResolver$EmptyBodyCheckingHttpInputMessage.<init>(AbstractMessageConverterMethodArgumentResolver.java:316)
		at org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodArgumentResolver.readWithMessageConverters(AbstractMessageConverterMethodArgumentResolver.java:193)
		at org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.readWithMessageConverters(RequestResponseBodyMethodProcessor.java:157)
		at org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.resolveArgument(RequestResponseBodyMethodProcessor.java:130)
		at org.springframework.web.method.support.HandlerMethodArgumentResolverComposite.resolveArgument(HandlerMethodArgumentResolverComposite.java:127)
		at org.springframework.web.method.support.InvocableHandlerMethod.getMethodArgumentValues(InvocableHandlerMethod.java:167)
		at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:134)
		at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:104)
		at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:892)
		at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:797)
		at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
		at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1039)
		at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:942)
		at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1005)
		at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:908)
		at javax.servlet.http.HttpServlet.service(HttpServlet.java:660)
		at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:882)
		at javax.servlet.http.HttpServlet.service(HttpServlet.java:741)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
		at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
		at ---自定义的---.CORSFilter.doFilter(CORSFilter.java:26)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
		at org.springframework.boot.actuate.web.trace.servlet.HttpTraceFilter.doFilterInternal(HttpTraceFilter.java:88)
		at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
		at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99)
		at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
		at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:92)
		at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
		at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:93)
		at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
		at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.filterAndRecordMetrics(WebMvcMetricsFilter.java:114)
		at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:104)
		at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
		at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200)
		at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118)
		at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
		at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
		at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
		at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
		at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:490)
		at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139)
		at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
		at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
		at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)
		at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:408)
		at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66)
		at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:853)
		at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1587)
		at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
		at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
		at java.lang.Thread.run(Thread.java:748)