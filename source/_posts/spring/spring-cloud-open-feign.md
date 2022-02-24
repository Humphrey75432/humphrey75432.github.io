---
title: Spring Cloud服务消费（Feign)
date: 2022-02-20 13:16:17
updated: 2022-02-20 13:16:17
tags: Spring Cloud
categories: 微服务架构学习
---

# Spring Cloud Open Feign

一套基于Netflix Feign实现的声明式服务调用客户端。它使得编写Web服务客户端变得更加简单。我们只需要通过创建接口并用注解来配置它即可完成对Web服务接口的绑定，并且支持可插拔的支持。除此之外，Spring Cloud Feign还扩展了对Spring MVC注解的支持，同时还整合了Ribbon和Eureka来提供负载均衡的HTTP客户端。

# 动手试试

下面我们演示一下Spring Cloud Open Feign的基本使用：

> 还是需要有eureka-server，eureka-client的支持

复制一个项目，命名为`eureka-consumer-feign`，在Gradle配置文件中写入下列依赖：

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

这里和普通的`eureka-client`不同的是，多了个`openfeign`的启动依赖；

## 修改应用主类

修改应用主类，通过`@EnableFeignClients`注解并且开启扫描Spring Cloud Feign客户端：

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaConsumerApplication {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerApplication.class, args);
    }

}
```

## 创建Feign客户端接口

我们实现一个客户端接口，并且使用`@FeignClient`指定接口所需要调用的服务名称，接口中定义的各个函数使用Spring MVC注解就可以用来绑定服务提供方的REST接口

```java
@FeignClient("eureka-client")
public interface DcClient {

    @GetMapping("/dc")
    String consumer();
}
```

## 修改Controller

通过定义的feign客户端来调用服务提供方接口：

```java
@RestController
public class DcController {

    @Resource
    private DcClient dcClient;

    @GetMapping("/consumer")
    public String dc() {
        return dcClient.consumer();
    }
}
```

# 总结

1. 在应用主类启用Open Feign服务
2. 定义Feign客户端接口，接口中的函数只需要使用Spring MVC注解就可以绑定；
3. 通过在控制器层定义feign客户端来调用服务提供方接口；

# 运行结果

跟之前一样，依次启动`eureka-server`, `eureka-client`以及`eureka-consumer-feign`来查看效果。
