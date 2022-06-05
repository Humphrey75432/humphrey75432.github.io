---
title: SpringBoot多数据源配置
date: 2022-05-25 22:24:52
updated: 2022-05-25 22:24:52
tags: Spring框架
categories: Spring
---

# 一、背景

最近负责一个项目，需要在SpringBoot中连接多个数据库。这篇博客记录一下Spring Boot集成多个数据库的配置用法。

# 二、环境设置

本文使用环境如下：

+ Spring Boot 2.6.7
+ MySQL Connector Java 8.0.28
+ 构建工具 Gradle
+ baomidou Mybatis-Plus 3.5.1
+ baomidou dynamic-datasource-spring-boot-starter 3.5.1

# 三、初始化项目

使用Spring Initializer新建Spring Boot项目，build.gradle文件配置如下：

```groovy
plugins {
    id 'org.springframework.boot' version '2.6.7'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'com.hhp.spring'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    maven { url 'https://maven.aliyun.com/repository/public/' }
    mavenLocal()
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'mysql:mysql-connector-java:8.0.28'
    compileOnly 'org.projectlombok:lombok'
    implementation 'com.baomidou:mybatis-plus-boot-starter:3.5.1'
    implementation 'com.baomidou:dynamic-datasource-spring-boot-starter:3.5.1'
    runtimeOnly 'mysql:mysql-connector-java:6.0.6'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}

```

application.yml文件如下：

```yaml
spring:
  datasource:
    dynamic:
      primary: story
      strict: false
      datasource:
        story:
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://localhost:3306/story?useSSL=false
          username: root
          password: <database password>
        test:
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://localhost:3306/test?useSSL=false
          username: root
          password: <database password>
```

新建两个数据库，配置如下：

story数据库

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : MySQL-Local
 Source Server Type    : MySQL
 Source Server Version : 80026
 Source Host           : localhost:3306
 Source Schema         : story

 Target Server Type    : MySQL
 Target Server Version : 80026
 File Encoding         : 65001

 Date: 25/05/2022 22:35:12
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for money
-- ----------------------------
DROP TABLE IF EXISTS `money`;
CREATE TABLE `money` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '用户名',
  `money` int NOT NULL DEFAULT '0' COMMENT '钱',
  `is_deleted` tinyint(1) NOT NULL DEFAULT '0',
  `create_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- ----------------------------
-- Records of money
-- ----------------------------
BEGIN;
INSERT INTO `money` VALUES (1, '对对对', 200, 0, '2022-05-25 21:54:25', '2022-05-25 21:54:25');
INSERT INTO `money` VALUES (2, '辅导费', 200, 0, '2022-05-25 21:54:33', '2022-05-25 21:54:33');
INSERT INTO `money` VALUES (3, '收到VS的v', 400, 0, '2022-05-25 21:54:43', '2022-05-25 21:54:43');
INSERT INTO `money` VALUES (4, '我带你们打', 9293, 0, '2022-05-25 21:54:56', '2022-05-25 21:54:56');
INSERT INTO `money` VALUES (5, 'fuck', 90, 0, '2022-05-25 21:55:03', '2022-05-25 21:55:03');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;

```

test数据库

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : MySQL-Local
 Source Server Type    : MySQL
 Source Server Version : 80026
 Source Host           : localhost:3306
 Source Schema         : test

 Target Server Type    : MySQL
 Target Server Version : 80026
 File Encoding         : 65001

 Date: 25/05/2022 22:35:24
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for money
-- ----------------------------
DROP TABLE IF EXISTS `money`;
CREATE TABLE `money` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '用户名',
  `money` int NOT NULL DEFAULT '0' COMMENT '钱',
  `is_deleted` tinyint(1) NOT NULL DEFAULT '0',
  `create_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- ----------------------------
-- Records of money
-- ----------------------------
BEGIN;
INSERT INTO `money` VALUES (1, '快快快', 98, 0, '2022-05-25 21:55:34', '2022-05-25 21:55:34');
INSERT INTO `money` VALUES (2, '都觉得', 20, 0, '2022-05-25 21:55:40', '2022-05-25 21:55:49');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;

```

# 四、编写业务代码

## 4.1 新建实体类

新建一个Money实体类，代码如下：

```java
@Data
@Accessors(chain = true)
@TableName("money")
public class Money {

    @TableId(value = "id", type = IdType.AUTO)
    private Integer id;

    private String name;

    private Long money;

    @TableField(value = "is_deleted")
    private Integer isDeleted;

    @TableField(value = "create_at")
    private Timestamp createAt;

    @TableField(value = "update_at")
    private Timestamp updateAt;

}
```

## 4.2 创建Mapper类以及Mapper文件

mapper类

```java
@Repository
public interface MoneyMapper extends BaseMapper<Money> {
}
```

mapper映射

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hhp.spring.mapper.MoneyMapper">
</mapper>

```

## 4.3 编写Service

编写MoneyService

```java
public interface IMoneyService extends IService<Money> {
}
```

编写StoryMoneyService，实现该接口

```java
@Service
@DS("story")
public class StoryMoneyServiceImpl extends ServiceImpl<MoneyMapper, Money> implements IMoneyService {
}
```

编写TestMoneyService，实现该接口

```java
@Service
@DS("test")
public class TestMoneyServiceImpl extends ServiceImpl<MoneyMapper, Money> implements IMoneyService {
}
```

注意到我们在Service上添加了注解`@DS`，这个注解来自于`dynamic-datasource-spring-boot-starter`。在我们进行dao层操作时可以根据不同的数据源灵活切换，从而做到查询不干扰。

除此之外，如果像将相同查询的写到同一个Service下管理，还可以这样实现：

```java
@Service
public class MoneyServiceImpl extends ServiceImpl<MoneyMapper, Money> implements IMoneyService {
    
    @DS("story")
    public List<Money> findByStoryIds(Collection<Long> ids) {
        return baseMapper.selectBatchIds(ids);
    }
    
    @DS("test")
    public List<Money> findByTestIds(Collection<Long> ids) {
        return baseMapper.selectBatchIds(ids);
    }
}
```

# 五、测试效果

直接在启动类编写如下代码，查看运行结果：

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@MapperScan("com.hhp.spring.mapper")
public class SpringBootDemoApplication {

    public SpringBootDemoApplication(StoryMoneyServiceImpl storyIMoneyService, TestMoneyServiceImpl testMoneyService) {
        List<Money> testMoneyList = testMoneyService.list();
        System.out.println("Test Money List: " + testMoneyList);

        List<Money> storyMoneyList = storyIMoneyService.list();
        System.out.println("Story Money List: " + storyMoneyList);
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringBootDemoApplication.class, args);
    }

}
```

程序运行结果如下：

```
2022-05-25 23:37:35.601  INFO 2871 --- [  restartedMain] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
 _ _   |_  _ _|_. ___ _ |    _ 
| | |\/|_)(_| | |_\  |_)||_|_\ 
     /               |         
                        3.5.1 
Test Money List: [Money(id=1, name=快快快, money=98, isDeleted=0, createAt=2022-05-25 21:55:34.0, updateAt=2022-05-25 21:55:34.0), Money(id=2, name=都觉得, money=20, isDeleted=0, createAt=2022-05-25 21:55:40.0, updateAt=2022-05-25 21:55:49.0)]
Story Money List: [Money(id=1, name=对对对, money=200, isDeleted=0, createAt=2022-05-25 21:54:25.0, updateAt=2022-05-25 21:54:25.0), Money(id=2, name=辅导费, money=200, isDeleted=0, createAt=2022-05-25 21:54:33.0, updateAt=2022-05-25 21:54:33.0), Money(id=3, name=收到VS的v, money=400, isDeleted=0, createAt=2022-05-25 21:54:43.0, updateAt=2022-05-25 21:54:43.0), Money(id=4, name=我带你们打, money=9293, isDeleted=0, createAt=2022-05-25 21:54:56.0, updateAt=2022-05-25 21:54:56.0), Money(id=5, name=fuck, money=90, isDeleted=0, createAt=2022-05-25 21:55:03.0, updateAt=2022-05-25 21:55:03.0)]
2022-05-25 23:37:35.942  WARN 2871 --- [  restartedMain] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2022-05-25 23:37:36.198  INFO 2871 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
```

# 六、Spring Boot + Mybatis Plus + JdbcTemplate多数据源配置

讲完上面基于baomidou的多数据源切换方案，我们理解一下通用的Spring Boot多数据源配置如何实现。

## 6.1 数据库准备

我们创建三个数据库来演示这个案例：
