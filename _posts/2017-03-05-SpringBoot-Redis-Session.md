---
layout: post
title:  "springboot redis实现session共享"
date:   2017-03-05 18:06:05
categories: Java
tags: redis tomcat session
---

* content
{:toc}
当创建分布式服务时会出现session共享的问题，即第一次访问的时候负载均衡会将请求分配到server1上，但是当第二次访问的时候，如果请求没有分配到server1上，那么用户的会话状态将丢失。下面给出了一种使用springboot整合redis的共享session的例子。





## 工具

- nginx-1.7.10
- redis-2.8

## 模拟分布式

使用nginx模拟一个简单的分布式的环境，在nginx.conf中添加
```
upstream  tomcat  {  
        server    127.0.0.1:8088  weight=1;
        server    127.0.0.1:9090  weight=2;  
} 
```
修改监听的端口，即所有的访问都通过83进行反向代理
```
 server {
        listen       83;
        server_name  localhost;
    }
```

## 创建一个springboot项目

在pom文件中加入

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-redis</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.session</groupId>
		<artifactId>spring-session-data-redis</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

## 配置redis和访问的端口

在application.properties中添加

```java
spring.redis.host=localhost
spring.redis.port=16001

server.port=9090

```

## 配置redis缓存session
```java
@Configuration
@EnableRedisHttpSession
public class RedisConfig {
}

```

## 编写一个测试的控制类
```java
@RestController
@RequestMapping(value = "/api/v1")
@PropertySource("classpath:application.properties")
public class HomeController {

    @Value("${server.port}")
    public String port;

    @RequestMapping(value = "/first", method = RequestMethod.GET)
    public Object firstResp(HttpServletRequest request) {
        Map<String, Object> map = new HashMap<>();
        request.getSession().setAttribute("url", request.getRequestURL());
        map.put("url", request.getRequestURL());
        map.put("port",port);
        return map;
    }

    @RequestMapping(value = "/sessions", method = RequestMethod.GET)
    public Object sessions (HttpServletRequest request){
        Map<String, Object> map = new HashMap<>();
        map.put("sessionId", request.getSession().getId());
        map.put("message", request.getSession().getAttribute("url"));
        map.put("port",port);
        return map;
    }

}

```

## 测试

复制一份相同的代码，然后区分访问端口为8088和9090

- 访问：http://localhost:83/api/v1/first

```json
{
	port: "9090",
	url: "http://tomcat/api/v1/first"
}
```

- 多次访问：http://localhost:83/api/v1/sessions

```json
{
	port: "9090",
	sessionId: "2e35353c-4d9e-4588-8257-43b54da9adc7",
	message: "http://tomcat/api/v1/first"
}

{
	port: "8088",
	sessionId: "2e35353c-4d9e-4588-8257-43b54da9adc7",
	message: "http://tomcat/api/v1/first"
}

```
虽然访问落到不同的服务下，但是每次获取的sessionId是相同的，即达到了session共享的目的。

## 目前缺点
- 目前nginx的配置，当一个服务挂掉的时候访问依然会落到这个服务商
- 目前的redis没有实现集群管理，当redis挂掉的时候，所有的session都将失效



## 参考资料
- [代码地址](https://github.com/MrDebuger/blog_src/tree/master/redis-session)

- [http://www.cnblogs.com/mengmeng89012/p/5519698.html](http://www.cnblogs.com/mengmeng89012/p/5519698.html)
- 可以参考github上的已有的集成方案[https://github.com/jcoleman/tomcat-redis-session-manager](https://github.com/jcoleman/tomcat-redis-session-manager)






