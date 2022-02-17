---
title: ES6 编码风格
date: 2022-02-17 22:21:12
updated: 2022-02-17 22:21:12
tags: ES6基础
categories: JavaScript
---

# 编程风格

## 块级作用域

### （1）使用`let`来替代`var`

原因之前讲过，`let`是ES6新增的关键字，和`var`一样可以用来声明变量，主要推荐用`let`的原因是：

- `let`和`var`完全等价，主要是`let`必须先声明再使用；
- `let`不存在变量提升；因此不会给程序带来副作用；



### （2）全局常量和线程安全

`let`和`const`主要先使用`const`，理由如下：

- `const`可以提醒阅读代码的人，这个变量不应该被改变；
- `const`符合函数式编程思想，运算不改变值，只是新建值；
- JavaScript编译器会对`const`进行优化，这样有利于提供程序的运行效率；



### （3）字符串

静态字符串一律使用单引号或引号，不使用双引号。动态字符串使用反引号；

```javascript
// bad
const a = "foobar";
const b = 'foo' + a + 'bar';

// acceptable
const c = `foobar`;

// good
const a = 'foobar';
const b = `foo${a}bar`;
```



### （4）解构赋值

遵循下列几个原则：

- 使用数组成员对变量赋值时，优先使用解构赋值；
- 函数的参数如果是对象成员，优先使用解构赋值；
- 函数返回多个值，优先使用对象的结构赋值，而不是数组的解构赋值；这样便于以后添加返回值，以及更改返回值的顺序；

例子如下：

```javascript
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr; // 这样它会拿出arr的第一个和第二个值
```

```javascript
// bad
function getFullName(user) {
    const firstName = user.firstName;
    const secondName = user.secondName;
}

// good
function getFullName(obj) {
    const {firstName, lastName} = obj;
}

// best
function getFullName({firstName, lastName}) {
    
}
```

```javascript
// bad
function processInput(input) {
    return [left, right, top, buttom];
}

// good
function processInput(input) {
    return {left, right, top, buttom};
}
const {left, right} = processInput(input);
```



### （5）对象

单行定义的对象，最后一个成员不以逗号结尾。多行定义的对象；最后一个成员以逗号结尾；

```javascript
// bad
const a = { k1: v1, k2: v2, };
const b = {
    k1: v1,
    k2: v2
};

// good
const a = { k1: v1, k2: v2 };
const b = {
    k1: v1,
    k2: v2,
};
```

对象尽量静态化，一旦定义就不得随意添加新的属性。如果添加属性不可避免，要使用`Object.assign()`方法；

```javascript
// bad
const a = {};
a.x = 3;

// if reshape unavoidable
const a = {};
Object.assign(a, {x: 3});

// good
cont a = { x: null };
a.x = 3;
```

如果对象的属性名是动态的，可以在创造对象的时候，使用属性表达式定义；

PS: [需要计算的内容]：这个语法就叫做计算属性，方括号中的内容需要通过动态计算得到；

```javascript
// bad
const obj = {
    id: 5,
    name: 'San Francisco'
};
obj[getKey('enabled')] = true;

// good
const obj = {
    id: 5,
    name: 'San Francisco',
    [getKey('enabled')]: true
};
```



### （6）数组

使用扩展运算符(...)拷贝数组：

```javascript
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i++) {
    itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```

使用`Array.from`方法将类似数组的对象转换为数组；

```javascript
const foo = document.querySelectorAll('.foo');
const nodes = Array.from(foo);
```



### （7）函数

立即执行函数可以写成箭头函数的形式

```javascript
(() => {
    console.log('Welcome to the Internet.')
})();
```

需要使用函数表达式的场合，尽量用箭头函数代替，因为这样更简洁，而且绑定了this

```javascript
// bad
[1, 2, 3].map(function(x){
    return x * x;
});

// good
[1, 2, 3].map((x) => {
    return x * x;
});

// best
[1, 2, 3].map(x => x * x);
```

箭头函数取代`Function.prototype.bind`，不再用self/_this/that绑定this。

```javascript
// bad
const self = this;
const boundMethod = function(...params) {
    return method.apply(self, params);
}

// acceptable
const boundMethod = method.bind(this);

// best
const boundMethod = (...params) => method.apply(this, params);
```

简单的、单行的、不会复用的函数，建议采用箭头函数。如果函数体比较复杂，行数较多，还是应该采用传统的函数写法。

```javascript
/* 用rest运算符代替arguments */
// bad
function concatenateAll() {
    const args = Array.prototype.slice.call(arguments);
    return args.join('');
}

// good
function concatenateAll(...args) {
    return args.join('');
}

/* 使用默认值语法设置函数参数的默认值 */
// bad
function handleThings(opts) {
    opts = opts || {};
}

// good
function handleThingd(opts = {}) {
    // ...
}
```





### （8）Map结构

只有模拟现实世界的实体对象，才使用Object。如果只是想要`key: value`的数据结构，使用Map结构。因为Map有内建的遍历机制。

```javascript
let map = new Map(arr);

for (let key of map.keys()) {
    console.log(key);
}

for (let value of map.values()) {
    console.log(value);
}

for (let item of map.entries()) {
    console.log(item[0], item[1]);
}
```



### （9）Class

总是使用Class来替代`prototype`的操作，因为Class写法更简洁，更易于理解；这个就不需要过多解释了。

```javascript
class Queue {
    constructor(contents = []) {
        this._queue = [...contents];
    }
    pop () {
        const value = this._queue[0];
        this._queue.splice(0, 1);
        return value;
    }
}
```

使用`extends`实现继承，因为这样更简单，不会破坏`instanceof`运算的危险，而且如果你是后端程序员，会更加理解这段逻辑想要表达的实际含义。

```javascript
class PeekableQueue extends Queue {
    peek() {
        return this._queue[0];
    }
}
```



### （10）模块

首先，Module语法是JavaScript模块的标准写法。坚持使用这种写法。使用`import`取代`require`。

```javascript
// bad
const moduleA = require('moduleA');
const func1 = moduleA.func1;
const func2 = moduleA.func2;

// good
import { func1, func2 } from 'moduleA';
```

使用`export`取代`module.exports`；

```javascript
// ES6 Style
import React from 'react';

const Breadcrumbs = React.createClass({
    render() {
        return <nav />;
    }
});

export default Breadcrumbs;
```



### （11）ESLint使用

ESLint是一个语法规则和代码风格的检查工具，可以用来保证写出语法正确、风格统一的代码。所有配置项都写在`.eslintrc`文件中，因此要启用eslint只需要在配置文件中开启相应的规则就可以。如果你是用脚手架工具新建的项目，一般都会集成eslint。
