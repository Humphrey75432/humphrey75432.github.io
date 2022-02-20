---
title: Spring Cloud服务消费（Ribbon)
date: 2022-02-20 09:40:26
updated: 2022-02-20 09:40:26
tags: Spring Cloud
categories: 微服务架构学习
---

# Spring Cloud Ribbon是什么

是基于Netflix Ribbon实现的一套**客户端负载均衡工具**，是一个基于HTTP和TCP的客户端负载均衡器。它可以通过在客户端中配置ribbonServerList来设置服务端列表去轮询访问达到负载均衡的作用。

# 具体操作

我们将继续利用之前的`eureka-server`作为服务注册中心，`eureka-client`作为服务提供者。还是跟之前一样，使用Spring Initializer来新建Spring Boot应用。

## 修改应用主类

为`RestTemplate`增加`@LoadBalanced`注解：

```java
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
    
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 修改Controller

去掉原来通过`LoadBalancerClient`选取实例和拼接URL的步骤，直接通过RestTemplate来发起请求；

```java
@RestController
public class DcController {
    
    @Resource
    private RestTemplate restTemplate;
    
    @GetMapping("/consumer")
    public String dc() {
        return restTemplate.getForObject("http://eureka-client/dc", String.class);
    }
}
```

# 有什么不同呢？

可以看到，我们去除了与`LoadBalancerClient`相关的逻辑之外，对于`RestTemplate`的使用，我们的第一个url参数有一些特别，直接采用了服务名的方式组成。

这里的实现细节是Ribbon实现了一个拦截器，能够在进行实际调用的同时，自动选取服务实例。并将实际请求的IP地址和端口替换成服务名，从而完成服务接口的调用。
