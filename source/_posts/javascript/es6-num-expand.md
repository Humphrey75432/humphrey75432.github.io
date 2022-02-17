---
title: ES6 数值扩展
date: 2022-02-17 15:05:52
updated: 2022-02-17 15:05:52
tags: ES6基础
categories: JavaScript
---

# 数值扩展

## 二进制和八进制数的表示方法

ES6提供了二进制和八进制数值的写法。二进制必须用`0b/0B`表示；八进制必须用`0o/0O`表示。示例如下：

```javascript
0B11111011 === 503 // true
0o767 === 503 // true
```

如果要将二进制/八进制数转换为十进制，需要使用`Number`方法：

```javascript
Number('0b111'); // 7
Number('0o10'); // 8
```

## 无穷数表示

ES6在原有Number的基础上新增了`Number.isFinite()`和`Number.isNaN()`两个方法。前者用于检查一个数值是否为有限数，后者检验一个数是否为`NaN`。例子如下：

```javascript
Number.isFinite(15); // true
Number.isFinite(0.8); // true
Number.isFinite(NaN); // false
Number.isFinite(Infinity); // false
Number.isFinite(-Infinity); // false

Number.isNan(NaN) // true
Number.isNan(15) // false
Number.isNan('15') // false
Number.isNan(9/NaN) // true
```

## `Number.parseInt()`和`Number.parseFloat()`

ES6将全局方法`parseInt()`和`parseFloat()`移植到Number对象上面，行为完全保持不变。

```javascript
Number.parseInt('12.34'); // 12
Number.parseFloat('123.45#'); // 123.45
```

## `Number.isInteger()`和`Number.EPSILON`

`Number.isInteger()`方法返回一个整型，因为JavaScript内部整数和浮点数使用的是同样的存储方法，因此3.0和3返回的都是整型3。

除此之外，ES6还新增了一个极小的常量`Number.EPSILON`。引入这个量的目的是判断当前浮点数计算的结果是不是我们期望的。由于浮点数计算存在精度误差，因此如果这个误差能够小于`Number.EPSILON`，我们就认为得到了正确结果。例子如下：

```javascript
function withInErrorMargin(left, right) {
    return Math.abs(left - right) < Number.EPSILON;
}

withInErrorMargin(0.1 + 0.2, 0.3) // true
withInErrorMargin(0.1 + 0.2, 0.3) // false
```

## 安全整数和`Number.isSafeInteger()`

JavaScript能够准确地表示的整数范围在 `-2^53` 至 `2^53` 之间（不包括两个端点），因此ES6中引入了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`两个常量，用来表示这个范围的上下限。

```shell
> Number.isSafeInteger(9007199254740993);
false
> Number.isSafeInteger(990);
true
> Number.isSafeInteger(9007199254740993 - 990);
true
>
```

所以在涉及大整型计算的过程中，需要同时校验运算数和运算结果是否为安全整型。例子如下：

```javascript
function trusty(left, right, result) {
    if (Number.isSafeInteger(left) &&
       Number.isSafeInteger(right) &&
       Number.isSafeInteger(result)) {
        return result;
    }
    throw new RangeError('Operation cannot be trusted!');
}

trusty(9007199254740993, 990, 9007199254740993 - 990)
// Operation cannot be trusted!

trusty(1, 2, 3);
// 3
```

## Math对象的扩展

### `Math.trunc()`

该方法用于去除一个数的小数部分，返回整数部分；对于非数值，会先将其转换为数值，再进行比较：

```javascript
Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4
Math.trunc(-4.9) // -4
Math.trunc(-0.1234) // -0
```

### `Math.sign()`

该方法用来判断一个数到底为正数、负数还是零。

```javascript
Math.sign(-5); // -1
Math.sign(5); // +1
Math.sign(0); // +0
Math.sign(-0); // -0
Math.sign(NaN); // NaN
Math.sign('foo'); // NaN
Math.sign(); // NaN
```

### `Math.cbrt()`

该方法用于计算一个数的立方根

```javascript
Math.cbrt(-1); // -1
Math.cbrt(0); // 0
Math.cbrt(1); // 1
Math.cbrt(2); // 1.2599210498948732
```

### `Math.clz32()`

JavaScript的整数使用32位二进制形式表示，`Math.clz32`方法返回一个数的32位无符号整数形式有多少个前导0。

```shell
> Math.clz32(0)
32
> Math.clz32(1)
31
>
> Math.clz32(1000)
22
> Math.clz32(1000)  
```

### `Math.imul()`

该方法返回两个数以32位带符号整数形式相乘的结果，返回的也是一个32位的带符号数。

```javascript
Math.imul(2, 4) // 8
Math.imul(-1, 8) // -8
Math.imul(-2, 2) // 4
```

### `Math.fround()`

该方法返回一个数的单精度浮点数形式。对于整数来说该方法没有任何效果，但是对于无法使用64个二进制位精确表示的小数，该方法会返回最接近这个小数的单精度浮点数。

```javascript
Math.fround(0) // 0
Math.fround(1) // 1
Math.fround(1.337) // 1.3370000123977661
Math.fround(1.5) // 1.5
Math.fround(NaN) // NaN
```

### `Math.hypot()`

该方法返回所有参数的平方和的平方根。例如注明的勾股定理（勾三股四弦五）

```javascript
Math.hypot(3, 4); // 5
Math.hypot(a, b, c); // sqrt(3^2 + 4^2 + 5^2);
```

### 对数方法

ES6新增了4个对数方法

- Math.expm1(x): 返回`e^x - 1`
- Math.log1p(x): 返回`1 + x`的自然对数，如果`x`小于-1， 返回`NaN`；
- Math.log10(): 返回以10为底的x的对数。如果`x`小于0，则返回`NaN`；
- Math.log2()：返回以2为底的x的对数。如果`x`小于0，则返回`NaN`;

### 三角函数方法

ES6新增了6个三角函数方法。

- Math.sinh(x)：返回`x`的双曲正弦
- Math.cosh(x)：返回`x`的双曲余弦
- Math.tanh(x)：返回`x`的双曲正切
- Math.asinh(x)：返回`x`的反双曲正弦
- Math.acosh(x)：返回`x`的反双曲余弦
- Math.atanh(x)：返回`x`的反双曲正切

### 指数运算

ES7新增了一个指数运算符`**`，目前Babel转码已经支持。这里不再赘述。
