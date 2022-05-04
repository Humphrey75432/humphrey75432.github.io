---
title: Golang入门教程
date: 2022-03-23 22:29:25
updated: 2022-03-23 22:29:25
tags: Go语言
categories: 后端开发
---

# 一、这篇文章的目的

按照以前的惯例，我们会把Golang的语法贴在这里，其实网上（包括官网）都有非常详细的Go语言教程，学习过程中可以多实践使用。这里我写一些使用中在语法上比较少涉及到的点。方便在使用过程快速回忆。当然golang本身也不复杂，上手也比较简单，如果有需要可以对照网上的教程重新温习。

# 二、golang基础

## 2.1 类型

+ 布尔值
  + **bool**
+ 字符串
  + **string**
+ 整型
  + **int** / **int8** / **int16** / **int32** / **int64**
  + **uint** / **uint8** / **uint16** / **uint32** / **uint64** / **uintptr**
+ 字节类型（uint8别名）
  + **byte**
+ Unicode类型（int32别名）
  + **rune**
+ 浮点类型
  + **float32** / **float64**
+ 复数类型
  + **complex64** / **complex128**

## 2.2 关键字汇总

golang关键字只有25个，非常容易掌握。写习惯面向对象的你可以很容易上手学习

+ 第一组
  + **break**（退出循环） / **case**（Switch语句） / **chan**（通道） / **const**（常量） / **continue**（跳过当前循环）
+ 第二组
  + **default**（Switch语句） / **defer**（先定义后执行） / **else**（分支） / **fallthrough**（Switch语句） / **for**（循环）
+ 第三组
  + **func**（函数定义） / **go**（开启Goroutine） / **goto**（流程） / **if**（条件） / **import**（导包）
+ 第四组
  + **interface**（接口或者泛型定义） / **map** / **package** / **range**（基于范围的遍历） / **return**（返回）
+ 第五组
  + **select**（在通道使用） / **struct**（结构体） / **switch**（分支语句） / **type**（定义类型） / **var**（声明变量）

## 2.3 包的导入导出

与Java不同的是，Go语言是通过名字大小写来判定当前函数/变量是否导出。如果是小写开头就意味着不导出；如果是大写开头就意味着是导出包。例如：

```go
// 未导出
math.pi

// 已导出
math.Pi
```

## 2.4 变量声明方式

与我们常用的C，C++，Java，JavaScript不同的是，Golang的变量类型是放在变量名的后面的。这点在写习惯了Java等编程语言的人来说，早期会感到不适应。但是随着你使用上习惯了，自然而然就能接受。具体如下：

```go
// 借助var关键字声明变量
var i int

// 自动推断类型，这里一定要注意，如果前面没有创建过变量i，必须使用":="符号
i := 34

// 多个变量初始化
const (
	i int
    j bool
    k string
)
```

## 2.5 类型转换

表达式`T(V)`将值`v`转换为类型`T`，这里提供一些关于数值转换的例子供你参考：

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```

或者使用更加简单的形式：

```go
i := 42
f := float64(i)
u := uint(f)
```

## 2.6 函数

跟Java不同的是，Go语言中的函数可以返回多个值，例如让你写一个交换两值的函数，在Java下你会这么写：

```java
public void swap(int x, int y) {
    System.out.printf("Before Swap is: %d, %d", x, y);
    int tmp = x;
    x = y;
    y = tmp;
    System.out.println("After Swap is: %d, %d", x, y);
}
```

但是在Go语言下，你只需要这样写就可以实现一样的效果了：

> 可以发现：在golang中函数的返回值也是放在函数尾部的，这和我们Java的语法存在不同，需要注意

```go
func swap(x, y int) (int, int) {
    return y, x
}
```

Go语言的返回值可以被命名，它们会被视作定义在函数顶部的变量，简而言之就是：没有参数的`return`语句返回已命名的返回值。也就是直接返回。例如：

> 直接返回语句应当仅用在短函数中，过长的函数代码反而会影响代码可读性

```go
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return
}
```

## 2.7 for循环

Go语言中只有一种循环结构，就是for，它的语法跟Java、C++很像。唯一的区别就是循环两侧不需要括号。例如：

```go
// 标准的for循环
for i := 0; i < 10; i++ {}

// Go语言中的While循环
for sum < 100 {}
```

除此之外，还可以使用for循环迭代切片和map

```go
var slice []int = []int{ 10, 20, 30, 40, 50 }
// 迭代切片
for index, value := range slice {
    fmt.Printf("索引为：%d，数值为：%d\n", index, value);
}

// 如果不想获取索引，可以使用"_"将其屏蔽
for _, value := range slice {
    fmt.Printf("索引为：%d，数值为：%d\n", index, value);
}

// 迭代map
var stock_map map[string]int = map[string]int{
	"iPhone 13 Pro": 345,
	"One Plus":      21,
	"Realme":        34,
}

for k, v := range stock_map {
	fmt.Printf("Key: %v, value: %d\n", k, v)
}
```

# 三、数组、切片和Map

## 3.1 数组

类型`[n]T`表示拥有n个T类型的值数组，语法如下：

```go
var a [10]int
```

具体实例如下所示：

```go
var a [2]string
a[0] = "hello"
a[1] = "world"

primes := [6]int{2, 3, 5, 7, 11}
```

## 3.2 切片

由于数组的大小是固定的，所以使用切片为数组元素提供动态大小、灵活的视角。在具体的开发实践中，切片比数组更加常用。具体语法如下：

```go
a[low:high]
```

它会选择一个半开区间，包括第一个元素但排除最后一个元素：

```go
// 代表a数组从下标1到3的元素（不包括4）
a[1:4]
```

切片并不存储任何数据，它仅仅是描述了底层数组中的一段，而且**更改切片中的元素会修改其底层数组**中对应的元素；除此之外，切片的下界默认为0，上界是该切片的长度。因此好好理解一下下列切片代表的含义：

```go
func main() {
	s := []int{2, 3, 5, 7, 11, 13}

	s = s[0:6] // 下界为0，上界为6
	fmt.Println(s)

	s = s[:6] // 下界默认为0，可以不写
	fmt.Println(s)

	s = s[0:] // 上界默认为6，可以不写
	fmt.Println(s)

	s = s[:] // 下界为0，上界为6
	fmt.Println(s)
    
    // 输出结果均为[ 2, 3, 5, 7, 11, 13 ]
}
```

切片有长度和容量，长度就是其所包含的元素个数，容量是从第一个元素开始数，到其底层数组元素的末尾个数。获取长度可以用`len(s)`，获取容量使用`cap(s)`。可以通过重新切片来扩展一个切片。给它提供足够的容量。

一个没有初始化的切片为nil切片，长度和容量均为0且没有底层数组（这就意味着有数据的切片存在一个底层数组，我们后面专门从底层讲讲切片）

创建切片可以使用内建的`make`函数来完成，也是用于创建动态数组的方式。

```go
a := make([]int, 5)
printSlice("a", a) // [0 0 0 0 0]

b := make([]int, 0, 5)
printSlice("b", b) // []

c := b[:2]
printSlice("c", c) // [0 0]

d := c[2:5]
printSlice("d", d) // [0 0 0]

func printSlice(s string, x []int) {
    fmt.Printf("%s len=%d, cap=%d %v\n", s, len(s), cap(s), x)
}
```

### 3.2.1 切片的切片

切片可以包含任何类型，甚至是包括其它切片，记录一下基本的使用：

```go
func main() {
    // board本身为一个切片，切片中每个元素又为一个切片
    board := [][]string{
        []string{"_", "_", "_"},
        []string{"_", "_", "_"},
        []string{"_", "_", "_"},
    }
}
```

### 3.2.2 向切片中追加元素

`Go`提供了内建的`append`函数，可以对已有切片进行元素的追加操作，具体使用事例如下所示：

```go
func main() {
    var s []int
    printSlice(s)
    
    s = append(s, 0)
    printSlice(s)
    
    s = append(s, 1)
    printSlice(s)
    
    s = append(s, 2, 3, 4)
    printSlice(s)
}

func printSlice(s []int) {
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s);
}
```

### 3.2.3 使用range循环切片

`for`循环的`range`形式可以遍历切片或映射，需要注意的是每次循环都会出现两个值，第一个值为当前元素的下标，第二个值为下标对应元素的副本，如果想要忽略下标，可以使用`_`符号；

```go
var pow = []int{1, 2, 4, 8, 16, 32, 64}

func main() {
    for i, v := range pow {
        fmt.Printf("2**%d = %d\n", i, v)
    }
}
```

## 3.3 Map

这种数据结构在熟悉不过了，如果你用过Java，那你应该对它很熟悉，介绍一下它的基本用法：

```go
type Vertex struct {
    Lat, Long float64
}

var m map[string]Vertex // 括号内为key的类型，括号外为value的类型，前面使用map关键字修饰

m = make(map[string]Vertex)
m["Bell Labs"] = Vertex{
    40.68433, -74.39967
}

//还可以忽略顶级类型名
var m = map[string]Vertex{
    "Bell Labs": {40.68443, -74.36883},
    "Google":    {37.43323, -122.08448},
}

// 修改也很简单
m := make(map[string]int)
m["Answer"] = 42 // 修改Key为Answer的值
delete(m, "Answer") // 删除Key为Answer的值
v, ok := m["Answer"] // 判断Answer的key是否存在
```

# 四、函数

## 4.1 函数也可以成为参数或者返回值

这也是Go语言面向函数式编程的核心思想，还有一个重要的概念叫闭包。例如：

```go
// compute为函数类型
// compute的函数参数为一个名为fn的函数，其接收两个函数参数， fn函数的返回值也为float64类型
// 返回值直接返回的也是函数类型
func compute(fn func(float64, float64) float64) float64 {
    return fn(3, 4)
}

func main() {
    hypot := func(x, y float64) float64 {
        return math.Sqrt(x*x + y*y)
    }
    fmt.Println(hypot(5, 12))
    fmt.Println(compute(hypot))
    fmt.Println(compute(math.Pow))
}
```

## 4.2 函数的闭包

Go函数可以是一个闭包，闭包是一个函数值，其引用了其函数体之外的变量，该函数也可以访问并赋予其引用的变量值。

```go
func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}

func main() {
    // 两个闭包都绑定在各自的sum变量上
    pos, neg := adder(), adder()
    for i := 0; i < 10; i++ {
        fmt.Println(pos(i), neg(-2*i))
    }
}
```

使用闭包实现斐波那契数列的算法可以这样写：

```go
func fibonacci() func() int {
    back1, back2 := 0, 1
    // 闭包返回的是具体的函数，所以这里也要返回函数
    return func() int {
        temp := back1
        back1, back2 = back2, (back1 + back2)
        return temp
    }
}

func main() {
    f := fibonacci()
    for i := 0; i < 10; i++ {
        fmt.Println(f())
    }
}
```

# 五、方法

## 5.1 基本概念

这里我们对照面向对象的思想，Go语言中没有“类”这个概念，所以我们一般可以借助结构体来定义方法。这个概念很重要，因为后面我们的很多面向对象的特性都是基于方法这个概念进行开展的。

```go
type Vertex struct {
    X, Y float64
}

// Vertex有一个方法，接收者为Vertex，可以理解为这个方法是Vertex的成员方法
func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

和一般的函数区别在于，方法在`func`关键字和方法名之间有一个参数列表，我们称之为`方法接收者`

## 5.2 指针接收者

可以为指针接收者声明方法，意味着对于某类型`T`，接收者可以是`*T`的文法。指针接收者的方法可以修改接收者指向的值。由于方法经常需要修改接收者，指针接收者比值接收者更加常用。

```go
type Vertex struct {
    X, Y float64
}

func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}
```

当然，你也可以使用函数来实现`Scale`方法，效果和使用指针接收器是一样的。

```go
type Vertex struct {
    X, Y float64
}

// 需要注意的是：如果涉及值修改，必须使用指针接收对象，否则值不会发生变化
func Scale(v *Vertex, f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}
```

这里可以得出一个结论：（在接收者参数类型上，值和指针满足交换律）

+ 以指针为接收者的方法被调用时，接收者既可以为值，又可以为指针；
+ 以值为接收者的方法被调用时，接收者既可以为值又可以为指针；

## 5.3 什么时候使用指针接收者

+ 方法能够修改其接收者指向的值
+ 避免每次调用方法调用进行大对象复制，特别是大型结构体，这样做更显得更加高效

# 六、接口(interface)

## 6.1 基本概念

接口类型是一组方法签名定义的集合，接口类型的变量可以保存任何实现了这些方法的值。例如：

```go
type I interface {
    M()
}

type T struct {
    S string
}

// 此方法表示类型T实现了接口I，但是不需要显式声明
func (t T) M() {
    fmt.Println(t.S)
}

func main() {
    var i T = T{"Hello"}
    i.M()
}
```

类型通过实现一个接口的所有方法来实现该接口，既然无需专门显式声明，也就没有`implements`关键字。

隐式接口从接口的实现解耦了定义，这样接口的实现可以出现在任何包中，无需提前准备。

## 6.2 接口值

接口也可以像其他值一样传递，可以用作函数的参数或者返回值，接口值保存了一个具体底层类型的具体值，接口值调用方法时会执行其底层类型的同名方法。

```go
type I interface {
    M()
}

type T struct {
    S string
}

func (t *T) M() {
    fmt.Println(t.S)
}

type F float64

func (f F) M() {
    fmt.Println(f)
}

func main() {
    var i I
    
    // 类似于面向接口编程
    i = &T{"Hello"}
    i.M()
    
    i = F(math.Pi)
    i.M()
}
```

nil接口值既不保存值也不保存具体类型，空接口`interface{}`可以保存任何类型的值，因此可以用于处理未知类型的值。例如：

```go
func main() {
    var i interface{}
    describe(i)
    
    i = 42
    describe(i)
    
    i = "Hello"
    describe(i)
}

func describe(i interface{}) {
    fmt.Printf("(%v, %T)\n", i, i)
}
```

## 6.3 类型断言

类型断言提供了访问接口值底层具体值的方式：

```go
t, ok := i.(T)
```

该语句断言的接口值`i`保存了具体类型`T`，并将其底层类型为`T`的值赋予变量`t`。为了判断一个接口值是否保存了一个特定类型，类型断言可以返回两个值：其底层值以及一个报告断言是否成功的布尔值。这里的语句和Map中的用法是一样的。例如：

```go
func main() {
    var i interface{} = "hello"
    
    s := i.(string)
    fmt.Println(s)
    
    s, ok := i.(string)
    fmt.Println(s, ok)
    
    f, ok := i.(float64)
    fmt.Println(f, ok)
    
    f = i.(float64)
    fmt.Println(f)
}
```

## 6.4 类型选择

类型选择是一种按照顺序从几个类型断言中选择分支的结构，类型选择一般与switch语句相似，不过类型选择中的case为类型（而非值），它们针对给定接口值所存储的值类型进行比较。其中`type`是固定写法。

```go
switch v := i.(type) {
    case T:
    	// v的类型为T
	case S:
    	// v的类型为S
	default:
    	// 没有匹配，v与i的类型相同
}
```

# 七、异常处理

Go语言使用`error`来表示错误状态（想想和Java中的有什么不同呢？），通常函数会返回一个`error`值，调用它的代码应当判断这个错误是否等于`nil`来处理错误，如果`error`为nil表示成功，否则表示失败；

```go
package main

import (
	"fmt"
	"time"
)

type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s", e.When, e.What)
}

func run() error {
	return &MyError{time.Now(), "it didn't work"}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err)
	}
}
```

# 八、Reader

io包指定了`io.Reader`接口，它表示从数据流的末尾进行读取，并且Go的标准库也包含了该接口的许多实现，包括文件、网络连接、压缩和加密。

## 8.1 字符读取

```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

# 九、并发

## 9.1 协程

协程（Goroutine）是由Go运行时管理的轻量级线程，Goroutine在相同的地址空间中运行，因此在访问共享内存时必须进行同步。`sync`包提供了这种能力，不过在Goroutine中并不常见。

```go
func say(s string) {
    for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
    }
}

func main() {
    go say("word")
    say("hello")
}
```

## 9.2 channel通道

通道是带有类型的管道，可以使用信道操作符`<-`来发送或者接收值（PS：“箭头”就是数据流的方向）

```go
ch <- v // 将v发送至信道ch
v := <- ch // 从ch接收值并赋予v
```

信道和切片一样，使用前必须创建，使用make函数来创建：

```go
ch := make(chan int)
```

默认情况下，发送和接收操作在另一端准备好之前都会阻塞，这就意味着Goroutine可以在没有显式锁或者竞态变量的情况下进行同步；

```go
func sum(s []int, c chan int) {
    sum := 0
    for _, v := range s {
        sum += v
    }
    c <- sum
}

func main() {
    s := []int{7, 2, 8, -9, 4, 0}
    c := make(chan int)
    go sum(s[:len(s)/2], c)
    go sum(s[len(s)/2:], c)
    x, y := <-c, <-c
    
    fmt.Println(x, y, x+y)
}
```

channel可以带缓冲区，有了缓冲区后，仅当信道的缓冲区填满后，向其发送数据时才会阻塞。当缓冲区为空时，接收方会阻塞。

```go
func main() {
    ch := make(chan int, 2)
    ch <- 1
    ch <- 2
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```

需要注意的是：仅当信道的缓冲区填满了以后，向其发送数据时才会阻塞，当缓冲区为空的时候，接受方会阻塞

发送者可以通过`close`关闭一个信道表示没有需要发送的值了，接收者可以通过为接收表达式分配第二个参数来测试信道是否被关闭；若没有值可以接收且信道已被关闭，那么执行结束后状态就会改成false。

> 1. 只有发送者才能关闭信道，接收者不能
> 2. 向一个已经关闭的信道发送数据会触发panic；
> 3. 信道与文件不同，通常情况下不需要关闭，只有在必须告诉接收者不再有需要发送的值才有必要关闭，例如**终止range循环**

例如：

```go
package main

import "fmt"

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
    // 当所有的值传输完后关闭通道
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```

## 9.3 select语句

select语句可以使得一个goroutine等待多个信道操作，select语句会阻塞某个分支可以继续执行为止，这时就会执行该分支。当多个分支都准备好时会随机选择一个执行。

```go
package main

import "fmt"

func fibonacci2(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci2(c, quit)
}

```

当`select`中的其他分支都没准备好时，`default`分支就会执行，为了尝试发送或者接收时不发生阻塞，可使用`default`分支：

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	tick := time.Tick(100 * time.Millisecond)
	boom := time.After(500 * time.Millisecond)
	for {
		select {
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}
```

