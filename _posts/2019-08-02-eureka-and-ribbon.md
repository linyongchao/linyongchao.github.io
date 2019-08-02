---
layout: post
title:  Eureka 注册服务及 Ribbon 负载均衡
date:   2019-08-02 15:36:54
categories: Java
---

* content
{:toc}

## EurekaServer

### pom.xml

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

### application.properties

	server.port=8765
	eureka.instance.hostname=localhost
	# 是否将自己注册到EurekaServer上，默认为true。由于当前应用就是EurekaServer，所以置为false
	eureka.client.register-with-eureka=false
	# 是否从EurekaServer获取注册信息，默认为true。单节点不需要同步其他EurekaServer节点数据，所以置为false
	eureka.client.fetch-registry=false
	eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/

### 启动类

	@EnableEurekaServer
	@SpringBootApplication
	public class EurekaServer {
	
	    public static void main(String[] args) {
	        SpringApplication.run(EurekaServer.class, args);
	    }
	}

## EurekaClient

### pom.xml

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

### application.properties

	server.port=8760
	spring.application.name=helloworld
	eureka.client.serviceUrl.defaultZone=http://localhost:8765/eureka/

### 启动类

	@EnableEurekaClient
	@SpringBootApplication
	public class EurekaClientOne {
	    public static void main(String[] args) {
	        SpringApplication.run(EurekaClientOne.class, args);
	    }
	}

### Controller

	@RestController
	public class TestController {
	
	    @Value("${server.port}")
	    String port;
	
	    @RequestMapping("/hello")
	    public String hello(String name) {
	        return port + " says:hello " + name;
	    }
	}

EurekaClient 需要至少两个才能看出 Ribbon 的负载均衡效果，所以将端口变一下启动两个 EurekaClient

## Ribbon

### pom.xml

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

### application.yml

	eureka:
	  client:
	    serviceUrl:
	      defaultZone: http://localhost:8765/eureka/
	server:
	  port: 8766
	spring:
	  application:
	    name: ribbon

### Ribbon

	@EnableDiscoveryClient
	@SpringBootApplication
	public class Ribbon {
	
	    public static void main(String[] args) {
	        SpringApplication.run(Ribbon.class, args);
	    }
	
	    @Bean
	    @LoadBalanced
	    public RestTemplate getRestTemplate() {
	        return new RestTemplate();
	    }
	}

### Service

	@Service
	public class TestService {
	
	    @Autowired
	    RestTemplate restTemplate;
	
	    public String hello(String name) {
	        return restTemplate.getForObject("http://helloworld/hello?name=" + name, String.class);
	    }
	}

### Controller

	@RestController
	public class TestController {
	
	    @Autowired
	    private TestService testService;
	
	    @RequestMapping("/hello")
	    public String hello(@RequestParam String name) {
	        return testService.hello(name);
	    }
	}

服务全部启动后，连续刷新 ```http://localhost:8766/hello?name=world``` 即可看到负载均衡效果