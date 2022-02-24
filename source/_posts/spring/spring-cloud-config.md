---
title: Spring Cloud配置中心
date: 2022-02-20 14:04:14
updated: 2022-02-20 14:04:14
tags: Spring Cloud
categories: 微服务架构学习
---

# 什么是Spring Cloud Config

是Spring Cloud团队创建的一个用来为分布式系统中的基础设施和微服务应用提供集中化的外部配置支持，它分为服务端与客户端两个部分。

+ 服务端：也称为分布式配置中心，是一个独立的微服务应用，用来连接配置仓库并为客户端提供获取配置信息、加密/解密信息等访问接口
+ 客户端：微服务架构中各个微服务应用或者基础设施，通过指定的配置中心来管理应用资源与业务相关的配置内容，并在启动时从配置中心获取和加载配置信息

# 有什么特点

+ 实现了对服务端和客户端中环境变量和属性配置的抽象映射；
+ 除了适用于Spring构建的应用还可以在任何其他语言运行的应用程序中使用；
+ 由于返回的配置结果为JSON，因此可以结合Git客户端工具管理和访问配置内容；

# 具体实例

下面构建一个基于Git存储的分布式配置中心，并对客户端进行改造，并让其从配置中心获取配置信息并绑定到代码中的整个过程。

## 构建配置中心

准备一个git仓库，创建一个`config-server`的Spring Boot项目，假设我们需要读取的配置中心为`config-client`，那么我们声明git仓库中该项目的默认配置文件为`config-client.yml`

默认环境

```yaml
info:
  profile: default
```

dev环境

```yaml
info:
  profile: dev
```

## 构建配置中心

借助Spring Cloud Config构建一个分布式配置中心非常简单，只需要三步：

+ 创建基础Spring Boot工程，命名为`config-server-git`，并且引入spring-cloud-config-server依赖；

+ 在Spring Boot主类上添加`@EnableConfigServer`注解，开启Spring Cloud Config的服务端功能；

+ 在`application.yml`添加配置服务的基本信息以及Git仓库信息：

  ```yaml
  spring:
    application:
      name: config-server
    cloud:
      config:
        server:
          git:
            uri: <这里的git仓库uri只能写到目录层，不能包括仓库名>
  server:
    port: 1201
  ```

  至此，使用一个通过Spring Cloud Config实现，并使用Git管理配置内容的分布式配置中心完成，可以先将应用启动起来，确保没问题再进行下面的操作。

  完成了这些准备工作之后，我们就可以通过浏览器、POSTMAN或CURL等工具直接来访问到我们的配置内容了。访问配置信息的URL与配置文件的映射关系如下：

  + /{application}/{profile}[/{label}]
  + /{application}-{profile}.yml
  + /{label}/{application}-{profile}.yml
  + /{application}-{profile}.properties
  + /{label}/{application}-{profile}.properties

  假设访问`http://localhost:1201/config-client/dev/master`，获得如下返回：

  ```json
  {
      "name": "config-client",
      "profiles": [
          "dev"
      ],
      "label": "master",
      "version": null,
      "state": null,
      "propertySources": [
          {
              "name": "http://git.oschina.net/didispace/config-repo-demo/config-client-dev.yml",
              "source": {
                  "info.profile": "dev"
              }
          },
          {
              "name": "http://git.oschina.net/didispace/config-repo-demo/config-client.yml",
              "source": {
                  "info.profile": "default"
              }
          }
      ]
  }
  ```

  ## 构建客户端

  接下来构建客户端，确保配置中心正常运作。创建一个Spring Boot应用，命名为`config-client`，引入下述依赖：

  ```groovy
  plugins {
      id 'org.springframework.boot' version '2.6.3'
      id 'io.spring.dependency-management' version '1.0.11.RELEASE'
      id 'java'
  }
  
  group = 'com.hhp.config'
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
      implementation 'org.springframework.cloud:spring-cloud-starter-config'
      // 从Spring Cloud 2.0.4开始起禁用了bootstrap，所以需要引入这个依赖才会生效
      implementation 'org.springframework.cloud:spring-cloud-starter-bootstrap'
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

  创建`bootstrap.yml`配置文件，制定获取配置文件中的config-server-git的位置：

  ```yaml
  spring:
    application:
      name: config-client
    cloud:
      config:
        uri: http://localhost:1201/
        profile: default
        label: master
  
  server:
    port: 2001
  ```

  上述配置参数与Git中存储的配置文件中各个部分的对应关系如下：

  + spring.application.name：对应配置文件规则中的`{application}`部分
  + spring.cloud.config.profile：对应配置文件规则中的`{profile}`部分
  + spring.cloud.config.label：对应配置文件规则中的`{label}`部分
  + spring.cloud.config.uri：配置中心`config-server`的地址

  另外，也可以通过修改config-client中的profile为dev观察加载配置的变化；
