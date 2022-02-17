---
title: ES6 函数扩展
date: 2022-02-17 15:07:14
updated: 2022-02-17 15:07:14
tags: ES6基础
categories: JavaScript
---

# 函数的扩展

## 函数参数的默认值

ES6以前函数参数是不能有默认值的，而ES6中对这个规则进行了修改了，函数参数也可以带默认参数。即直接写在参数定义的后面。

```javascript
function log(x, y = 'World') {
    console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```

这样的写法有两个好处：一个是阅读代码的人可以立刻意识到哪些参数是可以省略的，不需要查看函数体；其次有利于代码将来的优化。即便在未来的版本对外接口中，彻底拿掉这个参数也不会导致原有代码无法运行。

### 结合解构赋值

参数默认值当然可以使用前面章节提到的解构特性进行赋值了。

```javascript
function foo({x, y = 5}) {
    console.log(x, y);
}

foo({}) // undefined, 5
foo({x: 1}) // 1, 5
foo({x: 1, y: 2}) // 1, 2
foo() // Error
```

### 参数默认值的位置

一般情况下定义了默认值的参数应该是函数的尾部，这样比较容易看出来，如果是非尾部的参数设置默认值，实际上该参数是没法省略的。

```javascript
function (x = 1, y) {
    return [x, y];
}

f() // [1, undefined]
f(2) // [2, undefined]
f(, 1) // Error
f(undefined, 1) // [1, 1]

function (x, y = 5, z) {
    return [x, y, z];
}
f() // [undefined, 5, undefined]
f(1) // [1, 5, undefined]
f(1, ,2) // Error
f(1, undefinied, 2) // [1, 5, 2]
```

### length属性

函数的length属性将返回没有指定默认值的参数个数。也就是说：指定了默认值后，length属性将失真；

```javascript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
```

如果设置了默认值的参数不是尾参数，那么length属性也不再记入后面的参数了。

```javascript
(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1
```

### 作用域

如果参数默认值是一个变量，则该变量所处的作用域与其他变量的作用域规则是一样的。即先是当前函数的作用域，然后才是全局作用域。

```javascript
var x = 1;

function f(x, y = x) {
    console.log(y);
}

f(2) // 2
```

如果参数的默认值是一个函数，该函数的作用域是其声明时所在的作用域

```javascript
let foo = 'outer';

// func为一个默认匿名函数，返回值为变量foo
function bar(func = x => foo) {
    let foo = 'inner';
    console.log(func());
}

bar(); // outer
```

### 默认参数的用途一目了然

利用默认参数值，可以指定某一个参数不得省略，一旦省略就抛出一个错误；如果想省略这个参数，只要设置成`undefined`就行了。

```javascript
function throwIfMissing() {
    throw new Error('Missing parameters');
}

function foo(mustBeProvided = throwIfMissing()) {
    return mustBeProvided;
}

foo(); // Error: Missing Parameters
```

### rest参数

顾名思义：就是入参的个数是不确定的，跟Java中的多参数语法类似。这里就不再赘述了。

```javascript
function add(...values) {
    let sum = 0;
    
    for (var val of values) {
        sum += val;
    }
    
    return sum;
}

add(1, 2, 3, 4) // 10
```

## 扩展运算符

扩展运算符为三个点`...`，是将一个数组转化为用逗号分隔的参数序列，是rest参数的逆运算。

```javascript
console.log(...[1, 2, 3]) // 1 2 3

console.log(1, ...[2, 3, 4], 5) // 1 2 3 4 5

[...document.querySelectorAll('div')] // [<div>, <div>, <div>]
```

该运算符主要用于函数调用

```javascript
function push(array, ...items) {
    array.push(...items);
}

function add(x, y) {
    return x + y;
}

var numbers = [4, 38];
add(...numbers) // 42
```

### 扩展运算符的应用

#### （1）合并数组

```javascript
var arr1 = ['a', 'b'];
var arr2 = ['c'];
var arr3 = ['e', 'f'];

[...arr1, ...arr2, ...arr3] // ['a', 'b', 'c', 'e', 'f']
```

#### （2） 与解构赋值结合

```javascript
a = list[0], rest = list.slice(1)

[a, ...rest] = list

const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest // [2, 3, 4, 5]

const [first, ...rest] = []
first // undefined
rest // []

const [first, ...rest] = ['foo']
first // foo
rest // []
```

#### （3）函数返回值

JavaScript的函数只能返回一个值，如果需要返回多个值，只能返回数组或对象。扩展运算符提供了解决这个问题的一种办法：

```javascript
var dateFields = readDateFields(database);
var d = new Date(...dateFields);
```

#### （4）字符串

可以将字符串转换为真正的数组。因为扩展运算符可以很好地识别Unicode字符，因此最好都用扩展运算符；

```javascript
[...'hello']
// ['h', 'e', 'l', 'l', 'o']
```

#### （5）实现了Iterator接口的对象

任何Iterator接口的对象，都可以用扩展运算符转换为真正的数组，之前已经提过了，这里不再赘述

#### （6）Map和Set结构、Generator函数

承接第（5）点，Map和Set也部署了Iterator接口，所以也可以使用扩展运算符，Generator函数运行后返回的是一个遍历器对象，因此也可以使用扩展运算符。

```javascript
var go = function* () {
    yield 1;
    yield 2;
    yield 3;
};
[...go()] // [1, 2, 3]
```

## 严格模式

从ES5开始，函数内部可以使用严格模式，在ES2016中做了一点修改。规定只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数内部就不能显式设定为严格模式。

这样规定的原因是：函数内部的严格模式同时适用于函数体和函数参数。但是函数执行时会先执行函数参数代码，再执行函数体代码。因此只有从函数体代码之中才能知道参数代码是否应该以严格模式执行，但是函数参数代码又要先于函数体执行。所以这就导致了不合理的存在。

```javascript
function doSomething(value = 070) { // value = 070 先执行
    'use strict'; // 设置了严格模式
    return value; // 后执行
}
// Illegal 'use strict' directive in function with non-simple parameter list
```

### 但是我想要用严格模式限定时该怎么办呢？

```javascript
'use strict' // 全局严格模式

function doSomething(a, b = a) {}

// 由于函数参数部分会先执行：那很简单，将函数包在一个无参数的立即执行函数中
const doSomething = (function(){
    'use strict';
    return function(value = 42) {
        return value;
    }
});
```

## name属性

函数的`name`属性返回的是函数的名称。如果是将匿名函数赋给一个变量，在ES5中会返回空串，但是在ES6中会将变量名作为函数的名称。

```javascript
function foo () {}
foo.name // 'foo'

var func1 = function () {}

// ES5
func1.name // ""

// ES6
func1.name // "func1"
```

## 箭头函数

ES6中允许使用箭头`=>`来定义函数，定义效果和你使用`function`创建函数是等价的。

```javascript
var f = v => v;

// 等同于

var f = function(v) {
    return v;
};
```

如果箭头函数不需要参数或者需要多个参数，就是用圆括号来代表参数部分。

```javascript
var f = () => 5;
// 等同于
var f = function () {return 5;}

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
    return num1 + num2;
}
```

如果箭头函数的代码块部分超过一条语句，就是用大括号将它们围起来，必须在对象外面加上括号。

```javascript
var sum = (num1, num2) => {return num1 + num2;}

//大括号会解释为代码块，所以如果是用箭头函数返回对象，必须在对象外层加上大括号
var getTemplate = id => ({id: id, name: "Temp"});
```

箭头函数也可以与变量解构结合

```javascript
const full = ({ first ,last }) => first + ' ' + last;
// 等同于
function full(person) {
    return person.first + ' ' + person.last;
}
```

使用箭头函数需要注意几个地方：

- 函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象；
- 不可以当做构造函数；（`this`对象的指向是可变的，而在箭头函数中是固定的）
- 不可以使用`arguments`对象，如果要用，可以使用Rest参数代替；
- 不可以使用`yield`命令，因此箭头函数不可以用作Generator函数

```javascript
function foo() {
    setTimeout(() => {
        console.log('id:', this.id);
    }, 100);
}

var id = 21;
foo.call({ id: 42 });
// id: 42
```

## 绑定this

箭头函数可以绑定`this`对象，因此大大减少了显式绑定`this`对象的写法（`apply`、`call`、`bind`）。但是见图函数并不适用于任何场合，因此ES7提出了函数绑定运算符。语法为两个并排的冒号：`::`。双冒号左边是一个对象，右边是一个函数。该运算符会自动将左边的对象，作为上下文环境，绑定到右边的函数上面。

```javascript
foo::bar // bar.bind(foo)

foo::bar(...arguments); // bar.bind(foo, arguments)

const hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
    return obj::hasOwnProperty(key);
}
```

如果双冒号左边为空（对需要绑定的对象为空），则等于将该方法绑定在对象上面。

```javascript
var method = obj::obj.foo;
// 等同于
var method = ::obj.foo;

let log = ::console.log;
// 等同于
var log = console.log.bind(console);
```

由于双冒号运算符返回的还是原对象，因此可以采用链式写法。

```javascript
import { map, takeWhile, forEach } from "iterlib";

// 例1
getPlayers()
::map(x => x.character())
::takeWhile(x => x.strength > 100)
::forEach(x => console.log(x));

// 例2
let { find, html } = jake;
document.querySelectorAll("div.myClass");
::find("p")
::html("hahahh");
```

## 尾调用优化

尾调用是函数式编程中的一个重要概念：就是指某个函数的最后一步调用是另一个函数。例如下面的例子：函数f的最后一步是调用函数g。尾调用优化只在严格模式下才生效。

```javascript
function f(x) {
    return g(x);
}
```

下面三种情况都不属于尾调用：

```javascript
function f(x) {
    let y = g(x);
    return y;
}

function f(x) {
    return g(x) + 1;
}

function f(x) {
    g(x);
    return undefined;
}
```

尾调用不一定出现在函数尾部，只要是最后一步操作即可：

```javascript
function (x) {
    if (x > 0) {
        return m(x);
    }
    return n(x);
}
```

通常来说：函数调用会在内存中形成一个“调用记录”，也称为调用帧。保存调用位置和内部变量等信息。

由于尾调用是函数最后一步操作，因此不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧即可。

这里有一个前提条件：只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行“尾部调用优化”；

```javascript
function f() {
    let m = 1;
    let n = 2;
    return g(m + n);
}
f()

// 等同于
function f() {
    return g(3)
}
f();

// 等同于
g(3)
```

### 尾递归

基于上述尾部调用优化的特性：因此对于尾递归来说，由于只存在一个调用帧，因此永远也不会发生“栈溢出”错误。（注：将函数的多参数转换为单参数的范式成为”柯里化“）

```javascript
// 非尾递归写法
function factorial(n) {
    if (n === 1) return 1;
    return n * factorial(n - 1);
}

factorial(5) // 120

// 尾递归写法
function factorial(n, total = 1) {
    if (n === 1) return total;
    return factorial(n - 1, n * total);
}

factorial(5) // 120
```

还有斐波那契数列的例子：

```javascript
// 非尾递归写法
function Fibonacci(n) {
    if (n <= 1) { return 1 };
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}
Fibonacci(10); // 89
Fibonacci(1000); // 运行很慢

// 尾递归写法
function Fibonacci(n, ac1 = 1, ac2 = 1) {
    if (n <= 1) { return ac2; }
    return Fibonacci(n - 1. ac2, ac1 + ac2);
}

Fibonacci2(100) // 573147844013817200000
Fibonacci2(1000) // 7.0330367711422765e+208
```

可以看出“尾部调用优化”对于“尾递归"的意义重大，所以一些函数式编程语言将其写入了语言规格。换句话来说只要使用尾递归来实现就不会发生栈溢出，从而节省内存。

### 蹦床函数

由于尾部递归只能在严格模式下生效，理由如下：

正常模式下函数有两个变量，可以用于跟踪和记录函数调用：

- `func.arguments`：返回调用时的函数参数；
- `func.caller`：返回调用当前的函数；

尾部调用发生时，调用栈的记录会被改写，因此上面两个变量就会失真。而在严格模式下会禁用上面两个变量，因此尾部调用只能在严格模式下生效。

那么，如果我现在正常模式下实现尾部递归的写法又该如何做呢？可以将递归转换为循环执行。也就是俗称的“蹦床函数”：如下所示：`sum`函数每执行一次，都会返回自身的另一个版本。

```javascript
function trampoline(f) {
    while (f && f instanceof Function) {
        f = f();
    }
    return f;
}

function sum(x, y) {
    if (y > 0) {
        return sum.bind(null, x + 1, y - 1);
    } else {
        return x;
    }
}

trampoline(sum(1, 100000)); // 100001
```

但是上面的蹦床函数并不是真正的尾调用优化。下面这段代码来自官网教程。细细品位。

```javascript
function tco(f) {
    var value;
    var active = false; // 默认情况下该变量不激活
    var accumulated = [];
    
    return function accumulator() {
        accumulated.push(arguments);
        if (!active) {
            active = true; // 进入尾调用函数后激活
            while(accumulated.length) {
                value = f.apply(this, accumulated.shift());
            }
            active = false; // 调用完毕后关闭
            return value;
        }
    };
}

var sum = tco(function (x, y){
    if (y > 0) {
        return sum(x + 1, y - 1)
    } else {
        return x;
    }
});

sum(1, 10000) // 10001
```

## 函数参数的尾逗号

ES7将允许函数的最后一个参数有尾逗号。在此之前，不允许最后一个参数携带逗号;

```javascript
function clownsEveryWhere(
 param1,
 param2,
) {}
```
