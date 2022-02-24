---
title: 初识Spring Cloud
date: 2022-02-19 11:21:30
updated: 2022-02-19 11:21:30
tags: Spring Cloud
categories: 微服务架构学习
---

# Spring Cloud是什么？

一个基于Spring Boot实现的云应用开发工具，为基于JVM的云应用开发中涉及的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话以及集群状态管理等操作提供了一种简单的开发方式。

# 微服务架构

“微服务架构”这几年非常火热，以至于关注微服务架构相关的开源产品被反复提及（比如：Netflix和Dubbo），Spring Cloud也因Spring社区强大的知名度和影响力被广大架构师和开发者备受关注。

简单来说：微服务架构是将一个完整的应用从数据存储开始垂直拆分成多个不同的服务，每隔服务都能独立部署、独立维护、独立扩展，服务之间通过诸如RESTful API的方式相互调用。

# 服务治理

服务治理就是提供了微服务架构中各个微服务实例的快速上线或下线且保持各服务能正常通信的能力方案总称。

## 服务治理可以带来什么好处呢

+ **更高的可用性**

  支持动态的服务实例集群环境，任何服务实例都可以随时上线下线。并且当一个微服务实例不可用时，治理服务器可以将请求转发其他服务提供者，当新服务上线时，也能够快速分担服务调用请求；

  

+ **负载均衡**

  提供动态的负载均衡功能，可以将所有请求动态分布到其所管理的所有服务实例中进行处理；

  

+ **提升应用弹性**

  服务治理客户端定时从服务治理服务器中复制一份实例信息缓存到本地，即使当服务治理服务器不可用时，服务消费者也可以使用本地的缓存去访问相应的服务，而不至于中断服务。通过这种机制，极大提高应用的弹性；

  

+ **高可用性集群**

  可以构建服务治理集群，通过互相注册机制，将每隔治理服务器所管辖的服务信息列表进行交换，使服务治理拥有更高的可用性；

# Spring Cloud的基本使用

从上面对于微服务的治理和定义，我们搭建一个最基础和简单的微服务架构：

+ 服务注册中心
+ 服务提供方

这里我们使用Spring Cloud Eureka的基础服务来搭建，下面按照操作手册来做一下看看。

## 创建服务发现中心

使用IntelliJ IDEA创建一个Spring Boot项目，依赖管理工具选择Gradle。你可以使用Spring Initializer来生成项目：

<img src="https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/WX20220219-143445@2x.png" alt="s" style="zoom:50%;" />

生成项目后，我们找到项目目录的启动入口文件

```java
// 新增EnableEurekaServer注解，将该工程标识为一个服务注册中心
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

找到`application.properties`文件，将其更改为`application.yml`（只是习惯用YML文件），修改一下Eureka Server的配置如下：

```yaml
# 服务注册中心的名称
spring:
  application:
    name: eureka-server

# 服务注册中心监听的端口
server:
  port: 1001

# 暂时禁用服务端注册客户端的行为
eureka:
  instance:
    hostname: 127.0.0.1
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

到此，服务注册中心搭建完成，运行这个项目并且打开浏览器输入`http://localhost:1001/`就可以看到服务注册中心的基本页面了：

![](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/WX20220219-144301@2x.png)

## 创建服务提供方

如法炮制，我们通过Spring Initializer再创建一个Spring Boot项目，名为eureka-client，对应的gradle依赖如下：

主要提供的是：

+ spring-cloud-starter-netflix-eureka-client
+ spring-boot-starter-web

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

我们新建一个Rest接口：

> 需要注意的是：这里的DiscoveryClient会出现Netflix和Spring Cloud的，选择Spring Cloud的官方依赖

```java
@RestController
public class DcController {

    // 选择Spring Cloud的依赖包导入
    @Resource
    private DiscoveryClient discoveryClient;

    @GetMapping("/dc")
    public String dc() {
        String services = "Service: " + discoveryClient.getServices();
        System.out.println(services);
        return services;
    }
}
```

回到eureka-client的启动类，我们加上`@EnableDiscoveryClient`注解

```java
@EnableDiscoveryClient
@SpringBootApplication
public class EurekaClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```

打开eureka-client的`application.yml`文件，将下列的配置贴进去：

```yaml
spring:
  application:
    name: eureka-client
server:
  port: 2001
eureka:
  client:
    service-url:
      defaultZone: http://localhost:1001/eureka/
```

至此，服务提供方也创建成功。将服务注册中心和服务提供方都运行起来，打开注册中心可以看到eureka-client已经成功被注册。

![image-20220219162841266](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220219162841266.png)

# Spring Cloud Consul服务注册

使用Spring Cloud Consul实现，可以轻松将基于Spring Boot的微服务应用注册到Consul上，并通此实现微服务架构中的服务治理。

如果需要将上述的eureka-client转换成使用consul来，只需要按照下面2步操作即可：

+ 将spring-cloud-eureka-client依赖替换成spring-cloud-consul-discovery依赖
+ 修改application.yml中的配置

```yaml
spring:
  cloud:
    consul:
      host: 127.0.0.1
      port: 8500
```

其他配置不需要更改，并且由于Consul本身已经集成了服务端，所以不需要额外创建`eureka-server`这样的服务。直接通过下载consul服务端程序即可使用。此过程不再赘述。

更多关于Consul的使用指南，可以查看官方文档：https://www.consul.io/
