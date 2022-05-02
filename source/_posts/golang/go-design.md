---
title: Go设计哲学
date: 2022-05-01 23:45:50
updated: 2022-05-01 23:45:50
tags: Go语言
categories: 服务端开发
---

> Less is more，少即是多。这就是Go语言的设计哲学

这篇我们讲一些有关Go语言的设计哲学，这将有助你于学习Go语言的人理解这门语言的设计初衷以及它在实际应用领域中所能发挥的作用和局限性。

# 一、面向对象

## 1.1 如何理解面向对象

如果你是从C++，C#或者Java过来的程序员，你肯定疑惑过这个问题，Go语言中没有类似于`class`，`extends`以及`implements`的关键字，那是不是意味着Go语言就不支持面向对象了呢？

答案肯定是否定的，我们来对比一下Go和Java的区别：

| 语言特性 | Go                                    | Java                           |
| -------- | ------------------------------------- | ------------------------------ |
| 继承父类 | 使用结构体组合来实现类似于class的方式 | 使用`extends`关键字            |
| 实现接口 | 定义`interface`，方法签名实现即可     | 使用`implements`关键字实现接口 |

综上所述：Go语言使用了“组合优于继承”的方式实现面向对象的特性。我们用一个例子来演示说明：



## 1.2 简单的继承关系

一个人有姓名和年龄的基本属性，同时人又分男女。所以我们就用这个例子来演示：

### Java实现

```java
public class Person {
    private String name;
    private Integer age;
}

public class Male extends Person {
    private String gender = "M";
}

public class Female extends Person {
    private String gender = "F";
}
```

### Go实现

```go
type Person struct {
    name string
    age  int
}

type Male struct {
    gender string
    Person
}

type Female struct {
    gender string
    Person
}
```

可以看出，Go语言使用组合`Person`结构体的方式实现了对父类属性的重用，可以看出这种约定方式相比于Java来说要更加松散。



## 1.3 所以Go是面向对象语言吗？

按照Google官方的回答，Go语言是，也不是（——经典的薛定谔回答）；

主要从下面这几点来总结：

+ Go有类型和方法，并且允许面向对象的编程风格，没有层次概念（所以是OOP）；
+ Go中提供了“接口”的概念，以及将类型嵌套至其他类型中，以提供类似于类却不同于类的特性；
+ Go中的方法比C++和Java中的方法更加通用，它们不仅可以为结构体定义，也可以为任何内置类型定义。（这点就不像C++和Java那样具备强约束力了）；
+ 由于缺乏类型层次，所以Go语言中的对象要比其他编程语言中的对象更轻巧；

# 二、并发相关的设计哲学

## 2.1 可重入锁

在工程中使用互斥的根本原因是：为了保护不变量，也可以用于保护内、外部的不变量。

基于此，Go在互斥锁设计上会遵循下面这几个原则。如下：

+ 调用`mutex.Lock()`方法时，要保证这些变量的不变性保持，不会在后续过程中被破坏；
+ 调用`mutex.Unlock()`方法时，要保证程序不需要再依赖那些不变量；以及如果在互斥加锁期间破坏了它们，则需要确保已经恢复了它们；

结合官方例子来解释这个问题：

```go
func F() {
    mu.Lock()
    --- do more stuff ---
    G()
    --- do more stuff ---
    mu.Unlock()
}

func G() {
    mu.Lock()
    --- do more stuff ---
    mu.Unlock()
}
```

假如支持可重入锁，那么在`F()`方法执行完毕后就会跳到`G()`方法中，但是问题在于：**你完全不知道`F`和`G`方法在加锁后是不是做了什么事情**。从而导致破坏了不变量。

综上所述：Go语言不支持可重入锁，因为可重入锁会违反前面所提到的设计理念。因此要保证这些变量的不变性保持，不会在后续过程中被破坏。
