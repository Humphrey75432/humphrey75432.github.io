---
title: ES6 let和const的区别
date: 2022-02-17 10:50:28
updated: 2022-02-17 10:50:28
tags:
categories:
---

# let和const命令

## 基本用法

跟`var`的用法类似，都是用于定义变量，但是与`var`不同的是：`let`声明的变量仅在定义的区域起作用。如果你在IDE中使用`let`关键字必须先声明再使用，否则会直接报错无法通过ESLint编译。

```javascript
{
  let a = 123; // 变量a仅在这个代码区块有效
  var b = 'hello';
}

a // Undefined
b // hello
```

## 这有什么好处？

由于`let`声明的变量不会被覆盖，因此很适合在迭代器中做指针，看看下面这两段程序的结果会怎么样？

使用`var`做循环指针：每次迭代过程中指针`i`的值都会被覆盖

```javascript
var a = [];

for (var i = 0; i < 10; i++) {
  a[i] = function() {
    console.log(i);
  };
}

a[6](); // 10
```

使用`let`做循环指针：每次迭代过程中指针`i`的值都会指向当前值

```javascript
var a = [];

for (let i = 0; i < 10; i++) {
  a[i] = function() {
    cosole.log(i);
  };
}

a[6](); // 6
```

## 暂时性死区（Temporary Dead Zone）

只要块级作用域内存在`let`关键字，那么所声明的变量就与作用域牢牢地绑定在了一块。

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // 由于块区存在let绑定变量，因此这里报错显示变量未定义
  let tmp;
}
```

## 块级作用域

ES6中规定在块级作用域定义的变量只在所在块内才有效。块级可以嵌套。内层和外层可以定义同名的变量；

```javascript
{
  let name = 'hello' // 因为处在不同的块，所以不会报错
  {
    let name = 'world' // 内层定义跟外层不冲突
  }
}
```

## DO表达式

将块级作用域转换为表达式，这样就可以取到返回值，使用如下：如果不使用do表达式那么块内变量t在外层将无法取到；

```javascript
let x = do {
  let t = f();
  t * t + 1;
}
```

## 冻结对象

使用`const`关键字定义的变量跟`let`一样需要先定义再使用。但是常量并不意味着变量不能修改。看看下面的程序。

```javascript
const a = [];

a.push('Hello');
a // Hello

a.length = 0;
a // []

a = ['David'] // TypeError: Assignment to constant variable
```

如果需要冻结一个对象，可以使用`Object.freeze(obj)`，除了冻结对象，还需要冻结其属性。代码如下：

```javascript
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach((key, value) => {
    if (typeof obj[key] === 'object') {
      constantize(obj[key]);
    }
  });
};
```
