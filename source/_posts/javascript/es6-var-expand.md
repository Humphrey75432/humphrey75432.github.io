---
title: ES6 变量解构
date: 2022-02-17 10:53:06
updated: 2022-02-17 10:53:06
tags: ES6基础
categories: JavaScript
---

# 变量解构与赋值

## 基本用法

ES6允许按照一定的模式，从数组和对象中提取值，对变量进行赋值。写法如下：

```javascript
let [a, b, c] = [1, 2, 3];
a // 1
b // 2
c // 3

// 与下列写法等价
var a = 1;
var b = 2;
var c = 3;
```

本质上只有等号两边的模式匹配的情况下，解构变量才会成功，如果解构不成功，那么值就等于undefined；但是即便两边的模式不完全匹配也是可以解构成功的。即：等号右边的模式仅等于等号左边声明的一部分。例如：

```javascript
let [x, y] = [1, 2, 3];
x // 1
y // 2
```

## 适用范围

除了`var`变量可以解构赋值，还有`let`和`const`也可以解构，除此之外，一种数据结构只要其有Iterator接口都可以使用数组解构赋值。例如：

```javascript
let [a, b, c] = new Set({'aa', 'bb', 'cc'});
a // aa
b // bb
c // cc

const [d, e, f] = new Set({'dd', 'ee', 'ff'});
d // dd
e // ee
f // ff
```

使用Generator函数解斐波那契数列，内部使用解构赋值，每次循环过程中变量`a`和`b`都会重新赋值；

```javascript
function* fibs() {
  var a = 0;
  var b = 1;
  while(true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

var [first, second, third, fourth, fifth, sixth] = fibs();
sixth // 5
```

## 初始化

变量解构可以在声明前初始化，注意：如果解构右侧值不等于`undefined`那么初始化的值就不会生效。例如：

```javascript
let [x, y = 'y'] = [x, undefined];
y // y - 解构初始化成功

let [x, y = 'y'] = [x, null];
y // null - 解构初始化失败
```

如果解构默认值是一个表达式，那么只有在需要使用到表达式的时候才会赋值，否则默认值无效。

```javascript
function f() {
  console.log('aaa');
}

let [x = f()] = [1];
x // 1
// 由于x能在后面的数组中获取到数值1，因此默认值中的f()不生效；
```

## 对象的解构

除了可以对数组做解构，对象同样可以做解构，但是与数组有区别的是：对象解构只有在对象属性名相同的情况下才能解构成功，如下：

```javascript
let {foo, bar} = {foo: 'aaa', bar: 'bbb'}
foo // aaa
bar // bbb

let {baz} = {foo: 'aaa', bar: 'bbb'}
baz // undefined
```

如果变量名和属性名不一样必须写成下面这样才行；

```javascript
var {foo: baz} = {foo: 'aaa', bar: 'bbb'}
baz // aaa

let object = {first: 'hello', last: 'world'};
let {first: f, last: l} = object;
f // hello
l // world
```

解构也可以用于嵌套结构的对象

```javascript
var obj = {
  p: [
    'Hello',
    {y: 'world'}
  ]
};

var {p: [x, {y}]} = obj;
x // hello
y // world
// p 为模式而不是变量，因此也不会被赋值
```

## 解构已声明的变量

如果需要给已经声明的变量解构，一定要使用圆括号将其包裹起来，否则JavaScript引擎会将其解析为代码块。从而导致解构失败。ES6在解构变量时，除非会造成歧义必须使用圆括号围起来，否则不推荐使用圆括号；

```javascript
// Wrong syntax
var x;
{x} = {x: 1}

// Correct syntax
var x;
({x} = {x: 1})

// 还可以很方便地将现有对象的方法赋值到某个变量
let {log, sin, cos} = Math;
```

## 字符串的解构

字符串也可以解构，解构出来的结果是当作数组处理。程序代码如下：

```javascript
const [a, b, c, d, e] = 'hello';
a // 'h'
b // 'e'
c // 'l'
d // 'l'
e // 'o'

// 类似数组对象都存在一个length属性，也可以通过解构获得
let {length: len} = 'hello';
len // 5
```

## 数值和布尔值的解构赋值

解构赋值时，如果等号右边是数值和布尔值，则会先转换为对象；但是`undefined`和`null`无法转为对象，因此会解构失败；

```javascript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```

## 函数参数的解构赋值

函数参数也可以解构赋值

```javascript
function add([x, y]) {
  return x + y;
}

add([1, 2]); // 3

[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [3, 7]
```

函数参数的解构也可以使用默认值（这不是废话嘛）

```javascript
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

下面这种写法不是解构，而是给函数参数设置默认值

```javascript
function move({x, y} = {x: 0, y: 0}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

## 说了那么多，解构有什么用呢？

### 交换变量的值

```javascript
[x, y] = [y, x]
```

### 从函数返回多个值

```javascript
function example() {
  return [1, 2, 3];
}

var [a, b, c] = example();

function example() {
  return {
    foo: 1,
    bar: 
  };
}
  
var {foo, bar} = example();
```

### 函数参数的定义

```javascript
function f([x, y, z]) { ... }
f([1, 2, 3]);
```

### 提取JSON数据

```javascript
var jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let {id, status, data: number} = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

### 函数参数的默认值

```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function() {},
  cache = true,
  complete = function() {},
  crossDomain = false,
  global = true,
  // ... more config params
}) {
  // ... do stuff
};
```

### 遍历Map解构

```javascript
var map = new Map();
map.set('first', 'hello');
map.set('last', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}

// first is hello
// last is world

// 如果只想获取键名称，可以这样
for (let [key] of map) {
  // ... do something
}

// 只想获取键值
for (let [,value] of map) {
  // ... do something
}
```

### 输入模块的指定方法

```javascript
const { SourceMapConsumer, SourceNode } = require("source-map");

// 除此之外使用import也可以使用解构倒入指定方法
import { SourceMapConsumer, SourceNode } from 'source-map';
```
