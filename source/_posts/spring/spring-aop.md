---
title: 谈谈Spring AOP
date: 2022-03-04 20:55:35
updated: 2022-03-04 20:55:35
tags: Spring框架
categories: Spring
---

# 什么是Spring AOP

## 你可能遇到过这样的需求

假设要你统计一个方法执行的耗时情况，你会怎么做？

```java
public void runTask() {
    doSomething();
}
```

相信这个肯定难不倒你，那还不简单，在这个方法的执行前后两个时间戳，方法执行结束计算一下不是就能知道耗时了吗？

```java
public void runTask() {
    long start = Syste.currentTimeMills();
    doSomething();
    System.out.println("方法执行耗时：" + System.currentTimeMills() - start / 1000 + " s");
}
```

但是弊端也非常明显，直接将非业务功能和业务功能耦合在一起，成本非常大；稍有不慎就会出错；那么解决这个问题的方法是什么呢？就是AOP（面向切面编程）；

> 不用AOP，用你想到的设计模式怎么解决呢？
>
> + 将原对象通过组合包装到一个代理对象中；
> + 生成一个代理对象去包裹我们需要包含非业务的代码；
> + 在新创建的方法中再调用具体目标对象的方法；



## AOP的基本概念

### 定义

面向切面编程，是一种编程思想。用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装多个模块并命名为切面；面向切面编程使用的是动态代理实现的；

> 我们常用的有：打印日志、权限认证等场景；并且AOP需要解决下面几个问题：
>
> + 在什么地方进行切面操作
> + 需要知道切面操作的具体内容
> + 如果存在多个切面，怎样定义执行先后顺序



### 基本术语

AOP下包含几个重要的概念，需要理解：

+ **JointPoint（连接点）：**具体的切面点，可以是字段、方法、Spring具体表现为PointCut（切入点），仅作用在方法上；
+ **Advice（通知）：**在连接点进行的具体操作，如何进行处理增强，分为前置，后置，异常，最终和环绕；
+ **目标对象：**被AOP框架进行增强处理的对象，也被称为被增强对象；
+ **AOP代理：**AOP框架创建的代理对象，代理就是对被目标对象的加强，Spring中的代理既可以是JDK动态代理，也可以是cglib动态代理；
+ **Weaving：**将增强处理添加到目标对象中，创建一个被增强的对象过程；

> 一句话总结：在目标对象（target object）上的某些方法（jointpoint）添加不同种类的操作（通知、增强处理），最后通过某些方法（weaving、织入）实现一个新的代理目标对象；



# 怎样实现Spring AOP

Spring AOP的使用非常简单，我们以Spring框架为例：

## 引入依赖

比较关键的两个依赖是：

+ spring-aop
+ spring-aspects

```groovy
plugins {
    id 'java'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    maven { url 'https://maven.aliyun.com/repository/public/' }
    mavenLocal()
    mavenCentral()
}

dependencies {
    implementation 'org.springframework:spring-core:5.3.16'
    implementation 'org.springframework:spring-beans:5.3.16'
    implementation 'org.springframework:spring-tx:5.3.16'
    implementation 'org.springframework:spring-context:5.3.16'
    implementation 'org.springframework:spring-aop:5.3.16'
    implementation 'org.springframework:spring-aspects:5.3.16'
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.6.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine'
}

test {
    useJUnitPlatform()
}
```

## 创建目标Bean（被增强对象）

```java
@Component
public class TestBean {

    private String str = "Testing";

    public String getStr() {
        return str;
    }

    public void setStr(String str) {
        this.str = str;
    }
}
```

## 创建切面（Aspect）

这里需要注意几部分：

+ 切面要交给Spring容器管理，然后使用`@Aspect`注解来修饰；
+ 我们定义一个方法来存放切点，并且用`@Pointcut`注解修饰；
+ 创建Advice方法，也就是我们在被增强的对象执行前，后会做什么事情；这个就不再描述了；

```java
@Aspect
@Component
public class LogAspect {

    // 这里的意思是当前包下的所有对象中的方法都会被增强
    @Pointcut("execution(* com.hhp.spring..*(..))")
    private void allMethod() {}

    // 我们在这里获取被增强的类名和方法名，执行打印输出
    @Before("allMethod()")
    public void before(JoinPoint joinPoint) {
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = joinPoint.getSignature().getName();
        System.out.println(className + "." + methodName + "()开始执行...");
    }

    // 此处同上，不再赘述
    @After("allMethod()")
    public void after(JoinPoint joinPoint) {
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = joinPoint.getSignature().getName();
        System.out.println(className + "." + methodName + "()执行结束...");
    }
}
```

## 运行效果

我们编写一个启动类看看效果

```java
public class Application {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        TestBean testBean = (TestBean) ctx.getBean("testBean");
        System.out.println(testBean.getStr());
    }
}
```

控制台输出结果如下：

```shell
com.hhp.spring.beans.TestBean.getStr()开始执行...
com.hhp.spring.beans.TestBean.getStr()执行结束...
Testing
```



# 透过底层代码看Spring AOP

## 它是怎么工作的？

我们透过源代码来看看在配置类上添加`@EnableAspectJAutoProxy`注解就可以实现AOP功能，Spring在背后帮我们做了什么工作。

```java
// Registers an AnnotationAwareAspectJAutoProxyCreator against the current BeanDefinitionRegistry as appropriate based on a given @EnableAspectJAutoProxy annotation.
class AspectJAutoProxyRegister implements ImportBeanDefinitionRegistrar {
    	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
	}
}
```

**英文注释的意思是：**通过指定注释`@EnableAspectJAutoProxy`并根据当前的`BeanDefinitionRegistry`来注册一个`AnnotationAwareAspectJAutoProxyCreator`；所以我们在加了注解后就可以启用ProxyCreator去为我们创建AOP代理对象；

我们再去看`AnnotationAwareAspectJAutoProxyCreator`这个类它做了哪些工作：

```java
public class AnnotationAwareAspectJAutoProxyCreator extends AspectJAwareAdvisorAutoProxyCreator {
    @Override
    protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // 调用父类的方法初始化Bean工厂
        super.initBeanFactory(beanFactory);
    }
}
```

追踪代码到`AbstractAutowireCapableBeanFactory`如下：

```java
public abstract class AbstractAutowireCapableBeanFactory {
    
    protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
        Object wrappedBean = bean;
        if (mbd == null || !mbd.isSynthetic()) {
            // 执行beanPostProcessorsBeforeInitialization
            wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
            try {
                // 初始化Bean
                invokeInitMethod(beanName, wrappedBean, mbd);
            }
            
            if (mbd == null || !mbd.isSynthetic()) {
                // 执行beanPostProcessorsAfterInitialization
                wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
            }
            return wrappedBean;
        }
    }
}
```

我们看下通知是如何创建的，继续追踪源代码

```java
public abstract class AbstractAutoProxyCreator {
    protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
        // 省略其他不相关的代码...
        // Create proxy if we have advice
        Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
        if (specificInterceptors != DO_NOT_PROXY) {
            this.advisedBeans.put(cacheKey, Boolean.TRUE);
            Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
			return proxy;
        }
        
        this.advisedBeans.put(cackeKey, Boolean.FALSE);
        return bean;
    }
}
```

创建代理的源码如下：

```java
public abstract class AbstractAdvisorAutoProxyCreator {
    @Override
	@Nullable
	protected Object[] getAdvicesAndAdvisorsForBean(
			Class<?> beanClass, String beanName, @Nullable TargetSource targetSource) {

		List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
		if (advisors.isEmpty()) {
			return DO_NOT_PROXY;
		}
		return advisors.toArray();
	}
}
```

# 总结

我们用一张图再来回顾一下Bean的生命周期

![image-20220306123615086](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220306123615086.png)

用自己的话概括一下就是：

1. 生成并且注册BeanDefinition对象；
2. 初始化IoC容器并且加载BeanDefinition对象；
3. 寻找对应的BeanFactory去实例化Bean;（实例化阶段）
   1. 使用简单工厂 + 反射去推断构造函数进行实例化
4. 执行Bean的属性赋值动作
   1. 调用XXXAware接口回调方法；
   2. 调用初始化生命周期的三种回调方法；
   3. 假如Bean实现了AOP，会去创建动态代理对象；
5. 销毁Bean
   1. 销毁Bean的对象实例，仅在Spring容器关闭的过程中执行销毁；
