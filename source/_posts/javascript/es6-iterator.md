---
title: ES6 迭代
date: 2022-02-17 15:27:58
updated: 2022-02-17 15:27:58
tags: ES6基础
categories: JavaScript
---

# Iterator和for...of循环

## Iterator

### 基本概念

遍历器是是一种机制：它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署了Iterator接口，就可以完成遍历操作。

### 数据结构的默认Iterator接口

目的在于为所有数据结构，提供一种统一的访问机制，即`for...of`循环。因此一种数据结构只要部署了Iterator接口，我们就可以成为该数据结构是可遍历的。

ES6规定：默认的Iterator接口部署在数据结构的`Symbol.iterator`属性，也就是说：一个数据结构只要具有该属性，就可以认为是可遍历的。`Symbol.iterator`属性本身是一个函数，就是当前数据结构默认的遍历器生成函数。例子如下：

```javascript
const obj = {
    [Symbol.iterator] : function() {
        return {
            next: function() {
                return {
                    value: 1,
                    done: true
                };
            }
        };
    }
};

// obj对象是可遍历的，因为具有Symbol.iterator属性。
```

一个对象如果想要在`for...of`循环上调用Iterator接口，就必须在`Symbol.iterator`的属性上部署遍历器生成方法。

```javascript
class RangeIterator {
    constructor(start, stop) {
        this.value = start;
        this.stop = stop;
    }

    [Symbol.iterator]() { return this; }

    next() {
        var value = this.value;
        if (value < this.stop) {
            this.value ++;
            return {done: false, value: value};
        } else {
            return {done: true, value: undefined};
        }
    }
}

function range(start ,stop) {
    return new RangeIterator(start, stop);
}

for (var value of range(0, 3)) {
    console.log(value);
}
// 0
// 1
// 2
```

## 调用Iterator接口的场合

### （1）解构赋值

对数组和Set结构进行解构赋值时，会默认调用`Symbol.iterator`方法。

```javascript
let set = new Set().add('a').add('b').add('c');

let [x, y] = set;

let [first, ...rest] = set;
```

### （2）扩展运算符

扩展运算符默认也会调用`Symbol.iterator`接口，前面章节已经有很多例子，这里就不再展示了。

### （3）yield*

yield* 后面跟的也是一个可遍历结构，它会调用该结构的遍历器接口

```javascript
let generator = function* () {
    yield 1;
    yield* [2, 3, 4];
    yield 5;
};

var iterator = generator();

iterator.next()
{ value: 1, done: false }

iterator.next()
{ value: 2, done: false }

iterator.next()
{ value: 3, done: false }

iterator.next()
{ value: 4, done: false }

iterator.next()
{ value: 5, done: false }

iterator.next()
{ value: undefined, done: true }
```

### （4）其他场合

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都是调用了遍历器接口的。

## 字符串的Iterator接口

字符串是一个类似数组的对象，也原生具有Iterator接口。

```javascript
var someString = "hi";
typeof someString[Symbol.iterator]
// "function"

var iterator = someString[Symbol.iterator]();

iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }
```

## Iterator接口与Generator函数

```javascript
var myIterable = {};

myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};
[...myIterable] // [1, 2, 3]

// 或者采用下面的简洁写法

let obj = {
  * [Symbol.iterator]() {
    yield 'hello';
    yield 'world';
  }
};

for (let x of obj) {
  console.log(x);
}
// hello
// world
```

## 遍历器对象的return()，throw()

遍历器除了有`next`方法以外，还可以拥有`return`和`throw`方法。其中`next`方法是必须部署的，而`return`和`throw`却是可选的。

`return`方法的使用场景是：如果一个对象要在完成遍历前，需要清理或者释放资源，就需要部署。

`throw`方法主要结合Generator函数使用。这里不做过多解释。

```javascript
function readLinesSync(file) {
  return {
    next() {
      if (file.isAtEndOfFile()) {
        file.close();
        return { done: true };
      }
    },
    return() {
      file.close();
      return { done: true };
    },
  };
}
```

## for...of循环

一个数据结构只要部署了`Symbol.iterator`属性，就被视为具有iterator接口，就可以使用`for...of`循环遍历。也就是说`for...of`循环内部调用的是数据结构的`Symbol.iterator`方法。

### 数组

数组具备iterator接口，可以通过下面的代码证明：

```javascript
const arr = ['red', 'gree', 'blue'];

for (let v of arr) {
    console.log(v);
}

const obj = {};
obj[Symbol.iterator] = arr[Symbol.iterator].bind(arr);

for (let v of obj) {
    console.log(v);
}
```

### Set和Map结构

Set和Map结构也原生具有Iterator接口，可以直接使用`for...of`循环。前面有例子演示过，不再赘述。

### 类似数组的对象

类似数组的对象，包括：DON NodeList对象，arguments对象。都可以使用`for...of`来循环遍历。

```javascript
// 字符串
let str = "hello";

for (let s of str) {
  console.log(s); // h e l l o
}

// DOM NodeList对象
let paras = document.querySelectorAll("p");

for (let p of paras) {
  p.classList.add("test");
}

// arguments对象
function printArgs() {
  for (let x of arguments) {
    console.log(x);
  }
}
printArgs('a', 'b');
// 'a'
// 'b'
```

### 对象

对于普通对象，`for...of`不能直接使用，会报错；必须部署了iterator接口后才能使用。但是即便是这样：`for...of`依旧可以用来遍历键名。

```javascript
var es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};

for (let e in es6) {
  console.log(e);
}
// edition
// committee
// standard

for (let e of es6) {
  console.log(e);
}
// TypeError: es6 is not iterable
```

## `for...in`循环的缺点

`for...in`循环也是有缺点的：

- 数组的键名是数字，但是`for...in`循环是字符串作为键名的；
- 除了循环数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键；
- 某些情况下，`for...in`循环会以任何顺序遍历键名；

## `for...of`循环的优点

- 有着和for...in循环一样简洁的语法，但是没有for...in那些缺点；
- 不同于forEach方法，可以结合return, continue和break使用；
- 提供了遍历所有数据结构的统一操作接口；
