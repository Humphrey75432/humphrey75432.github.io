---
title: Go设计哲学
date: 2022-05-01 23:45:50
updated: 2022-05-01 23:45:50
tags: Go语言
categories: 后端开发
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

## 2.2 并发读写

Go语言中的slice和map是非线程安全的。可以通过下面两个例子来说明：

### （1）Slice并发

```go
func main() {
    var s []string
    go func(){
        for i := 0; i < 9999; i++ {
            s = append(s, i)
        }
    }()
}
fmt.Printf("Len is %d\n", len(s))
```

这段程序每次执行都会发现结果不一样，但确定的是，slice的长度无论如何都达不到9999，很显然，一个协程写的数据被其他的协程覆盖了。导致其出现非线程安全的情况。

很显然，Go语言的设计者在一开始设计切片就希望不会在并发情况下使用，按照Java的逻辑，对容器中元素的维护需要对其索引进行修改，这对于Go语言这种追求极简方式的设计哲学格格不入。所以官方就没有对其实现。

### （2）Map的并发

我们再来看一下Map对并发情况的支持如何：

```go
func main() {
    s := make(map[string]string)
    go func() {
        for i := 0; i < 99; i++ {
            s["Name"] = "Time"
        }
    }()
}
fmt.Printf("Total is %d times\n", len(s))
```

这段程序直接执行会报错。错误如下：

```shell
fatal error: concurrent map writes

goroutine 52 [running]:
runtime.throw({0x10a5618?, 0x0?})
	/usr/local/go/src/runtime/panic.go:992 +0x71 fp=0xc000123748 sp=0xc000123718 pc=0x102f0f1
runtime.mapassign_faststr(0x0?, 0x0?, {0x10a2b55, 0x4})
	/usr/local/go/src/runtime/map_faststr.go:212 +0x39c fp=0xc0001237b0 sp=0xc000123748 pc=0x101009c
main.main.func1(
/golang_learning/concurrent/unsafe2.go:10 +0x30 fp=0xc0001237e0 sp=0xc0001237b0 pc=0x108ad30
runtime.goexit()
	/usr/local/go/src/runtime/asm_amd64.s:1571 +0x1 fp=0xc0001237e8 sp=0xc0001237e0 pc=0x105a521
created by main.main
```

可以看出，Go语言设计者对Map也不支持并发操作。因为Go官方不支持Map读写。原因如下：

+ map的典型使用场景不需要从goroutine中进行安全访问；
+ 非典型场景：map可能只是一些更大的数据结构已经同步计算的一部分；
+ 性能场景考虑：若只是为少数程序添加安全性，导致Map的所有处理都需要Mutex操作，将会极大降低程序的性能

综上所述，Go官方认为map不需要支持并发访问。

那如果我非要使用并发访问Map应该怎么做呢？可以使用Go语言支持的`sync.Map`：

```go
var m sync.Map

func main() {
	data := []string{"Hello1", "Hello2", "Hello3", "Hello4"}
	for i := 0; i < 4; i++ {
		go func(i int) {
			m.Store(i, data[i])
		}(i)
	}
	time.Sleep(time.Second)

	v, ok := m.Load(0)
	fmt.Printf("Load: %v, %v\n", v, ok)

	m.Delete(1)

	v, ok = m.LoadOrStore(1, "Fuck2")
	fmt.Printf("Load: %v, %v\n", v, ok)

	m.Range(func(key, value any) bool {
		fmt.Printf("Range: %v, %v\n", key, value)
		return true
	})
}
```

