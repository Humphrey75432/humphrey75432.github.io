---
title: Spring Cloud服务消费（基础）
date: 2022-02-19 21:02:44
updated: 2022-02-19 21:02:44
tags: Spring Cloud
categories: 微服务架构学习
---

# 使用LoadBalancerClient

这是一个负载均衡客户端的抽象定义，负载均衡的好处在于：

+ 提供动态的负载均衡功能，可以将所有请求动态分布到其所管理的所有服务实例中进行处理；

所以在分布式系统设计中，负载均衡可以用来实现系统的高可用、集群扩容等功能。负载均衡也分为“客户端负载均衡”和“服务端负载均衡”两种模式。

我们的Spring Cloud Eureka属于客户端负载均衡的情况，所以这里我们用一个具体的例子来演示客户端负载均衡的具体实现。

> 此Demo需要结合前面已有的eureka-server和eureka-client一起看效果

# 创建消费服务

还是按照之前的套路，使用Spring Initializer新建一个Spring Boot工程，叫`eureka-consumer`。

## 添加依赖

对应的gradle文件如下：

```groovy
plugins {
    id 'org.springframework.boot' version '2.6.3'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'com.hhp.registry'
version = '0.0.1'
sourceCompatibility = '1.8'

repositories {
    maven { url 'https://maven.aliyun.com/repository/public/' }
    mavenLocal()
    mavenCentral()
}

ext {
    set('springCloudVersion', "2021.0.1")
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
    }
}

tasks.named('test') {
    useJUnitPlatform()
}
```

## 配置服务发现

打开`application.yml`文件，贴入下列配置：

```yaml
spring:
  application:
    name: eureka-consumer
server:
  port: 2101
eureka:
  client:
    service-url:
      defaultZone: http://localhost:1001/eureka/
```

找到Spring Boot启动入口类，在上面加上`@EnableDiscoveryClient`注解，这里我们把RestTemplate带入我们的配置中：

```java
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

## 创建服务消费接口

我们新建一个接口来消费eureka-client服务

```java
@RestController
public class DcController {

    @Resource
    private LoadBalancerClient loadBalancerClient;

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer")
    public String dc() {
        ServiceInstance serviceInstance = loadBalancerClient.choose("eureka-client");
        String url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort() + "/dc";
        System.out.println(url);
        return restTemplate.getForObject(url, String.class);
    }
}
```

可以看出：

1. 我们注入了`LoadBalancerClient`和`RestTemplate`，并在`/consumer`接口中实现；
2. 通过`LoadBalancerClient.choose`选择出`eureka-client`的服务实例；
3. 然后通过`ServiceInstance`服务的信息拼接出`/dc`接口的详细地址；
4. 利用`RestTemplate`对象实现对服务提供者接口的调用；

# 最终效果

将`eureka-server`，`eureka-client`和`eureka-consumer`三个服务同时运行起来，然后访问`http://localhost:2101/consumer`。可以看到当服务器请求过程中，真正调用的是`eureka-client`，而`eureka-consumer`会把请求通过网络分发出去。
