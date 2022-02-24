---
title: Spring知识汇总
date: 2022-02-24 19:45:04
updated: 2022-02-24 19:45:04
tags: Spring框架
categories: Spring
---

# 1. 为什么使用Spring

+ IoC技术，容器帮忙管理依赖对象，不需要自己创建和管理依赖对象，更轻松实现程序解耦；
+ 事务支持，使得操作事务更加方便；
+ 提供面向切面的编程，可以更方便地处理某一类的问题；
+ 更方便集成其他框架，比如Mybatis和Hibernate；



# 2. 什么是AOP

面向切面编程，通过预编译的方式和运行期动态代理实现程序功能的统一维护的技术；简单来说就是统一处理“切面”类问题的编程思想，比如统一处理日志，异常等；



# 3. 什么是IoC

IoC是Spring的核心，对于Spring框架来说，由Spring来负责控制对象的生命周期和对象之间的关系；

控制指的是当前对象对内部成员的控制权；控制反转指的是这种控制权不由当前对象管理，由其他（类、第三方容器）来管理；



# 4. Spring模块

+ spring core：框架基础，提供IoC和依赖注入
+ spring context：构建于core封装包基础上的context封装包，提供一种框架式的对象访问方法
+ spring dao：提供JDBC抽象层
+ spring aop：提供面向切面的编程实现，可以自定义拦截器、切点
+ spring web：提供针对web开发的集成特性，例如文件上传，利用servlet listeners进行ioc容器初始化和针对web的ApplicationContext；
+ spring web mvc：提供了web应用的Mode-View-Controller的实现；



# Spring常见的注入方式

+ setter属性注入
+ 构造方法注入
+ 注解方式注入



# Spring的bean是线程安全的吗？

Spring的bean默认是单例模式，并且Spring框架没有对单例bean进行多线程的封装处理；但由于大部分时候spring bean是无状态的，因此某种程度上来说bean也是安全的，如果bean有状态的话，就需要开发者自己保证线程安全。

最简单的就是改变bean的作用域，将singleton变更为prototype，这样请求bean就相当于new Bean了，就可以保证线程安全；

+ 有状态就是有数据存储功能
+ 无状态就是不会保存数据



# Spring支持的Bean作用域

+ singleton：Spring IoC容器只存在一个bean实例，bean以单例模式存在，是系统默认
+ prototype：每次从容器调用bean都会创建一个新的实例；
+ Web环境下的作用域
  + request：每次http请求都会创建一个bean；
  + session：同一个http session共享一个bean实例；
  + global-session：用于portlet容器，每个portlet有单独的session，globalsession提供一个全局性的http session;

**注意：**使用prototype作用域需要慎重考虑，因为频繁创建和销毁session会带来很大的性能开销；



# Spring自动装配Bean的方式

+ no：默认值，没有自动装配，使用显式bean引用进行装配
+ byName：根据bean名称注入对象依赖项；
+ byType：根据类型注入对象依赖项；
+ 构造函数：通过构造函数注入依赖，需要设置大量参数；



# Spring事务实现方式

+ 声明式事务：基于XML配置文件和注解方式（在类上添加`@Transaction`注解）；
+ 编码方式：提供编码的形式管理和维护事务；



# Spring事务的隔离

+ ISOLATION_DEFAULT：使用底层数据库的隔离级别，数据库设置什么我就用什么；
+ ISOLATION_READ_UNCOMMITED：未提交读，最低隔离级别，事务未提交时，就可被其他事务读取（出现幻读、脏读、不可重复读）；
+ ISOLATION_READ_COMMITED：提交读，一个事务提交后才能被其他事务读取到（会造成幻读、不可重复读），sql server 的默认级别；
+ ISOLATION_REPEATABLE_READ：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（会造成幻读），mysql 的默认级别；
+ ISOLATION_SERIALIZABLE：序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。



# Spring MVC的运行流程

+ spring mvc 先将请求发送给 DispatcherServlet。
+ DispatcherServlet 查询一个或多个 HandlerMapping，找到处理请求的 Controller。
+ DispatcherServlet 再把请求提交到对应的 Controller。
+ Controller 进行业务逻辑处理后，会返回一个ModelAndView。
+ Dispathcher 查询一个或多个 ViewResolver 视图解析器，找到 ModelAndView 对象指定的视图对象。
+ 视图对象负责渲染返回给客户端。



# Spring MVC的组件

+ 前置控制器 DispatcherServlet。
+ 映射控制器 HandlerMapping。
+ 处理器 Controller。
+ 模型和视图 ModelAndView。
+ 视图解析器 ViewResolver。



# @Autowired和Resource的区别

## Autowired注解

+ Autowired有一个问题：当一个类型有多个bean值的时候会造成无法选择具体注入哪一个的情况，此时需要配合@Qualifier使用；
+ Autowired为Spring提供的注解；

## Resource注解

+ 由J2EE提供；
+ 默认byName自动注入；
+ 同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛异常；
+ 指定了name，从上下文查找名称匹配的bean进行装配，找不到抛异常；
+ 指定了type，从上下文找到类似匹配的唯一bean进行装配，找不到或者找到多个，都会抛异常；

总结：使用@Resource可以减少代码和Spring之间的耦合



# 什么是Spring Boot

为Spring服务，用来简化Spring应用的初始化搭建以及开发过程；

Spring Boot有下列好处，这也是为什么现在流行广泛的原因；

+ 配置简单
+ 独立运行
+ 自动装配
+ 无代码生成和xml配置
+ 提供应用监控
+ 容易上手
+ 提升开发效率



# Spring Boot的核心配置文件

Spring Boot核心的两个配置文件：

+ bootstrap.yml：有ApplicationContext加载，比application优先加载，且bootstrap里面的属性值不能被覆盖；
+ application.yml：用于spring boot项目的自动化配置；



# Spring Boot实现热部署

+ 使用Devtools启动热部署，添加devtools库，在配置文件中把spring.devtools.restart.enabled设置为true;
+ 使用IntelliJ IDEA编译器，勾上自动编译或手动编译；
