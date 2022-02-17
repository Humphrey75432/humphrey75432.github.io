---
title: ES6 数组扩展
date: 2022-02-17 15:06:02
updated: 2022-02-17 15:06:02
tags: ES6基础
categories: JavaScript
---

# 数组扩展

## Array.from()

用于将类似数组的对象和可遍历的对象（部署了Iterator接口）转换为数组。例子如下：

```javascript
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};

// ES5
var arr1 = [].slice.call(arrayLike);

// ES6
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```

实际应用中，常见的类似数组对象是DOM操作返回的NodeList集合以及函数内部arguments对象。Array.from都可以将它们转为真正的数组。

```javascript
let ps = document.querySelectorAll('p');
Array.from(ps).forEach(function (p) {
    console.log(p);
});

function add(a, b) {
    var args = Array.from(arguments);
    console.log(args);
    return a + b;
}

add(1, 2);
// [1, 2]
// 3

// 字符串和Set都部署了Iterator接口，所以可以用Array.from生成数组，这点和Java很像
Array.from('hello'); // ['h', 'e', 'l', 'l', 'o']
let nameSet = new Set(['a', 'b']);
Array.from(nameSet) // ['a', 'b']
```

除此之外，扩展运算符`...`也可以将某些数据结构转换为数组（注：扩展运算符跟Java中的多参数表达式功能类似）。扩展运算符背地也是调用Iterator接口，如果对象没有部署这个接口，则无法完成转换；

```javascript
function foo () {
    var args = [...arguments];
}

[...document.querySelectorAll('div')]
```

## Array.of()

Array.of用于将一组值，转换为数组；这个方法的主要目的是弥补Array()方法的不足。因为参数个数的不同，会导致Array()的行为有差异。

```javascript
Array.of(3, 11, 8); // [3, 11, 8]
Array(3, 11, 8); // [3, 11, 8]
Array(3); // [, , ,] 长度为3的空数组
Array.of(3); // [3] 长度为1的数组，只有一个元素3
```

## 数组实例的copyWithin()

在当前数组的内部，将指定位置的成员复制到其他位置，然后返回当前数组。也就是说该方法会修改当前数组。

```javascript
Array.prototype.copyWithin(target, start = 0, end = this.length)
// target（必需）：从该位置开始替换数据
// start（可选）：从该位置开始读取数据，默认为0，如果为负值则表示倒数；
// end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示倒数；

[1, 2, 3, 4, 5].copyWithin(0, 3); // [4, 5, 3, 4, 5]
```

## 数组实例find()和findIndex()

`find()`方法用于找出第一个符合条件的数组成员。其第一个参数是回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员。如果没找到符合条件的成员则返回`undefined`。

`findIndex()`方法的作用与`find()`方法非常类似。这里就不再赘述。可以看出新增的两个方法极大地弥补了`indexOf()`方法的不足。

```javascript
[1, 4, -5. 10].find((n) => n < 0); // -5

[1, 5, 10, 15].find(function(value, index, arr){
    return value > 9;
}); // 10

[1, 5, 10, 15].findIndex(function(value, index, arr){
    return value > 9;
}) // 2（正好是元素10对应的数组索引）
```

## 数组实例fill()

`fill()`可以使用给定值来填充数组，因此在初始化数组元素的过程中非常方便。

初次之外，fill()还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置

```javascript
['a', 'b', 'c'].fill(7); // [7, 7, 7]

new Array(3).fill(7); // [7, 7, 7]

['a', 'b', 'c'].fill(7, 1, 2); // ['a', '7', 'c'] 用7填充，从1号位开始，到2号位结束
```

## 数组实例entries(), keys()和values()

ES6新增了上述的3个方法，英语遍历数组。它们都是返回一个遍历器对象，可以使用`for...of`循环遍历，唯一区别是`keys()`是对键名的遍历、`values()`是对键值得遍历，`entries()`是对键值对的遍历。

```javascript
for (let index of ['a', 'b'].keys()) {
    console.log(index)
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
    console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
    console.log(index, elem);
}
// 0 'a'
// 1 'b'
```

## 数组实例includes()

`Array.prototype.includes`方法返回一个布尔值，表示某个数组是否是否已经包含给定的值，与字符串的`includes`方法类似。该方法属于ES7，但是Babel转码器已经支持。

```javascript
[1, 2, 3].includes(2); // true
[1, 2, 3].includes(4); // false
[1, 2, NaN].includes(NaN); // true
```

该方法还有第二个参数表示搜索的起始位置，默认为0 ，如果为负数则表示倒数位置。

## 数组的空位

数组的某一个位置没有任何值。比如`Array`构造函数返回的数组都是空位。注意：空位不等于`undefined`

```javascript
var arr = new Array(3);
0 in arr // false
undefined in arr // false

var arr = new Array(3).fill(undefined)
0 in arr // true
```

在ES6中会明确将空位转换为`undefined`， `copyWithin()`会将空位一起拷贝，`fill()`会将空位当做正常位置，循环也会迭代空位。总之ES6中会保留空位。由于空位的处理规则不统一，因此应当避免出现空位。

```javascript
[...['a',,'b']]
// ['a', undefined, 'b']
```
