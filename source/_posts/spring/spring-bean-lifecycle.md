---
title: Spring Bean的生命周期学习
date: 2022-03-04 12:53:11
updated: 2022-03-04 12:53:11
tags: Spring框架
categories: Spring
---

# 什么是Spring Bean

用一句话来讲解，Spring Bean就是由Spring的IoC容器所创建出来的对象，就称之为Bean；

IoC（Inverse of Control）是Spring中最核心的理念。要理解什么是IoC容器，我们先从一个最简单的例子讲起：



## Java中如何创建一个对象？

相信你只要做过Java肯定都知道，用`new`关键字来创建一个对象，然后JVM会根据对象的构造函数去成这个对象，从而完成对象的创建。

```java
SomeObject someObject = new SomeObject();
```

除此之外，我们使用反射也可以创建出一个Java对象

```java
Class.forName("类的所在包的全路径").newInstance();
```

这两种方法的本质都是：调用了我们对象的构造函数来实现的，并且需要注意的是：整个对象创建的过程都是有程序员自己手动完成的。



## 在Spring中创建的对象跟手动创建的对象有什么区别的？

也许你发现了，在使用Spring框架的过程中，我们从来都不需要显式地去自己创建对象，而是通过下面这样的代码来实现：

```java
// 创建Spring IoC容器
ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
// 也可以是这样子
context = new AnnotationConfigApplicationContext(AppConfig.class);

// 从Spring IoC容器中获取一个Bean
UserService userService = (UserService) context.getBean("userService");
```

可以看到，我们在指定的Spring IoC容器中通过名称就可以拿到我们的Bean对象；

这说明，在Spring内部通过IoC设计思想已经帮助我们创建好了我们想要用的对象，这种设计思想就叫做“控制反转”。

控制反转提现在两个概念：

+ 控制：对象创建的控制权不再是程序员，而是Spring IoC容器；
+ 反转：指的就是创建对象的这种控制权交给了第三方容器来实现；

Spring使用了“依赖注入”的方式实现了这种机制。所以**总结下来：控制反转是设计思想，依赖注入是具体实现。并且依赖注入是控制反转的一种实现方式；**



## 如何理解依赖注入呢

我们再通过一个朴素的例子来说明这个问题：

```java
// 定义一个AppConfig类
@ComponentScan(basePackages = "当前类所在的包")
public class AppConfig {}

// UserService Bean
@Component
public class UserService {
    @Autowired
    private OrderService orderService;
}

// OrderService
@Component
public class OrderService {}

// Main方法
public class Application {
    
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        UserService userService = (UserService) ctx.getBean("userService");
    }
}
```

我们通过Debug来看下userService对象里面有什么内容？

![image-20220304131346171](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220304131346171.png)

可以看到我们在`UserService`中依赖了`OrderService`，在Spring Ioc容器中我们的`OrderService`也顺便被实例化了。这就是Spring IoC容器中依赖注入帮我们实现的业务逻辑；

## 这是如何做到的呢？

很简单，Spring中有个BeanFactory负责创建对象，创建完对象我们可以通过类名拿到对应的对象，这个是反射实现的。因此对于Spring IoC的机制也很简单。就是：

简单工厂（BeanFactory + 动态代理）



# Spring Bean的生命周期

需要记住的是，Spring Bean的生命周期只有四个阶段：

+ **实例化**
  + 借助反射推断构造函数进行实例化；使用的是实例工厂和静态工程；
+ **属性赋值**
  + 解析自动装配（byName，byType，构造函数，none以及@Autowired）
  + 此步骤也是依赖注入的实现机制
  + 除此之外，赋值阶段会解决循环依赖的问题；
+ **初始化**
  + 调用XXXAware接口（BeanNameAware，BeanClassLoaderAware以及BeanFactoryAware）
  + 调用初始化生命周期的三种回调方式；（@PostConstruct， InitializeBean以及init-method属性）
  + 假设Bean实现了AOP，那么会去创建动态代理；
+ **销毁**
  + Spring关闭容器时调用并且销毁Bean对象；



## 1. Bean对象创建的过程中

我们对应到源码中看下具体的流程：

```java
public abstract class AbstractAutowiredCapableBeanFactory {
    
    // 省略其他无关代码
    protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
        BeanWrapper instanceWrapper = null;
        if (instanceWrapper == null) {
            // 实例化阶段
            instanceWrapper = createBeanInstance(beanName, mbd, args);
        }
        Object bean = instanceWrapper.getWrappedInstance();
        Object exposedObject = bean;
        try {
            // 属性赋值阶段
            populateBean(beanName, mbd, instanceWrapper);
            // 初始化阶段
            exposedObject = initializeBean(beanName, exposedObject, mbd);
        } catch (Throwable ex) {
            throw (BeanCreationException) ex;
        }
    }
}
```

回到createBean方法我们继续看：

```java
public abstract class AbstractAutowiredCapableBeanFactory {
    
    protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
        RootBeanDefinition mbdToUse = mbd;
        try {
            // 实例化之前
            Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        }
        
        try {
            // 创建Bean的位置
            // 包括实例化、属性赋值和初始化阶段
            Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        }
    }
}

@Nullable
protected Object resolveBeforeInstantiation(String beanName, RootDefinition mbd) {
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        if (!mbd.isSynthetic() && hasInstantiationAwareBeforePostProcessors()) {
            // 省略下列代码
        }
    }
}

protected boolean hasInstantiationAwareBeforePostProcessors() {
    return !getBeanPostProcessorCache().instantiationAware.isEmpty();
}

// Method in AbstractBeanFactory
BeanPostProcessorCache getBeanPostProcessorCache() {
    BeanPostProcessorCache bpCache = this.beanPostProcessorCache;
    if (bpCache == null) {
        bpCache = new BeanPostProcessorCache();
        for (BeanPostProcessor pb : this.beanPostProcessors) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                // Ignore other code
            }
        }
    }
}
```

最后追踪的代码过程中，我们找到了`InstantiationAwareBeanPostProcessor`接口，看下它的实现：

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
    @Nullable
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}
    
    default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}
}
```

找到属性赋值的方法，源码如下：

```java
public abstract class AbstractAutowireCapableBeanFactory {
    protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
        // Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
		// state of the bean before properties are set. This can be used, for example,
		// to support styles of field injection.
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
				if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
					return;
				}
			}
		}
    }
}
```

对此可以总结两点：

+ postProcessBeforeInstantiation在doCreateBean之前调用，也就是在bean实例化之前调用的，英文源码注释解释道该方法的返回值会替换原本的Bean作为代理，这也是Aop等功能实现的关键点。
+ postProcessAfterInstantiation在属性赋值方法内，但是在真正执行赋值操作之前，返回值为boolean为false时可以阻断属性赋值阶段；



## 2. 无所不能的Aware

Aware之前的名字就是可以拿到什么资源，例如`BeanNameAware`可以拿到BeanName，一次类推。调用时机需要注意：所有Aware方法都是在初始化阶段之前就调用；

## 第一组

+ BeanNameAware
+ BeanClassLoaderAware
+ BeanFactoryAware



## 第二组

+ EnvironmentAware
+ EmbeddedValueResolverAware
+ ApplicationContextAware

可以通过源码看下具体的内容：

```java
public abstract class AbstractAutowiredCapableBeanFactory {
    
    protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
        // 调用第一组的三个Bean开头的Aware
        invokeAwareMethods(beanName, bean);
        
        Object wrappedBean = bean;
        // 调用第二组的几个Aware接口
        // BeanPostProcessor调用点1
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
        // InitializingBean的调用点
        invokeInitMethods(beanName, wrappedBean, mbd);
        // BeanPostProcessor调用点2
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
        return wrappedBean;
    }
}
```

这里区别记忆：

+ BeanxxxAware接口是在代码中直接调用的；
+ ApplicationContext相关的Aware接口是通过在BeanPostProcessor下的postProcessBeforeInitialization()实现的；



# 简单的两个生命周期接口

剩下的两个接口就比较简单，一个是`initializingBean`，一个是`DisposableBean`；前者对应生命周期的初始化阶段；后者对应生命周期的销毁阶段；



# 总结

Spring Bean的生命周期分为`4个阶段`和`多个扩展点`。扩展点又可以分为`影响多个Bean`和`影响单个Bean`：

## 4个阶段

+ 实例化 Instanitiation
+ 属性赋值 Populate
+ 初始化 Initialization
+ 销毁 Destruction



## 多个扩展点

+ 影响多个Bean
  + BeanPostProcessor
  + InstantiationAwareBeanPostProcessor
+ 影响单个Bean
  + Aware Group 1
    + BeanNameAware
    + BeanClassLoaderAware
    + BeanFactoryAware
  + Aware Group 2
    + EnvironmentAware
    + EmbeddedValueResolverAware
    + ApplicationContextAware
+ 生命周期
  + InitializingBean
  + DisposableBean
