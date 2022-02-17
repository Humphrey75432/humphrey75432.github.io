---
title: ES6 对象扩展
date: 2022-02-17 15:08:22
updated: 2022-02-17 15:08:22
tags: ES6基础
categories: JavaScript
---

# 对象扩展

## 属性的简洁表示法

ES6允许直接写入变量和函数，作为对象的属性和方法。这样书写更加简洁。

```javascript
var foo = 'bar';
var baz = {foo};
baz // {foo: "bar"}

// 等同于
var baz = {foo: foo};
```

除了属性可以简写，方法也可以简写

```javascript
var o = {
    method() {
        return "Hello";
    }
};

// 等同于

var o = {
    method: function() {
        return "Hello";
    }
};
```

适用于函数返回值，写起来会非常简洁和方便

```javascript
function getPoint() {
    var x = 1;
    var y = 10;
    return {x, y};
}

getPoint() // {x: 1, y: 10}
```

在使用Common JS中，输出的代码非常适合使用这种简洁写法

```javascript
var ms = {};

function getItem(key) {
    return key in ms ? ms[key] : null;
}

function setItem(key, value) {
    ms[key] = value;
}

function clear() {
    ms = {};
}

module.exports = { getItem, setItem, clear }
// 等同于
module.exports = {
    getItem: getItem,
    setItem: setItem,
    clear: clear
}
```

## 属性名表达式

JavaScript语言定义对象的属性，有如下种方法。

```javascript
obj.foo = true; // 直接使用标识符作为属性名

obj['a' + 'bc'] = 123; // 使用表达式作为属性名
```

ES6允许字面量定义对象时，用方法二作为对象的属性名，即将表达式放在括号内

```javascript
let propKey = 'foo';

let obj = {
    [propKey]: true,
    ['a' + 'bc']: 123
}
```

这里再举一个例子：

```javascript
var lastWord = 'last word';

var a = {
    'first word': 'hello',
    [lastWord]: 'world'
};

a['first Word'] // hello
a[lastWord] // World
a['last word'] // World
```

注意：属性名表达式如果是一个对象，默认情况下会自动将对象转换为字符串。因此属性名表达式和简洁表示法不可以同时使用。一定要注意。

## 方法的`name`属性

函数的`name`属性返回函数名。对象方法也是函数，因此也有`name`属性。

```javascript
var person = {
    sayName() {
        console.log(this.name);
    },
    // get为取值函数，存值用set
    get firstName() {
        return "Nicholas";
    }
};

person.sayName.name // sayName
person.firstName.name // get firstName
```

有两个特例：如果是`bind`函数，函数名返回`bound` + 函数名称；如果是`function`关键字构造的函数（匿名函数），`name`属性值返回`anonymous`。

## `Object.is()`

该方法用来比较两个值是否严格相等，与严格比较运算符`===`的作用一致。不同之处有两个：+0不等于-0；另一个是NaN等于自身。

```javascript
+0 === -0 // true
NaN === NaN // false

Object.is(+0, -0); // false
Object.is(NaN, NaN) // true
```

## `Object.assign()`

### 基本用法

用于将对象合并，将源对象的所有可枚举属性赋值到目标对象。如果目标对象与源对象由同名属性，或多个源对象由同名属性，则后面的属性会覆盖前面的属性。

```javascript
var target = { a: 1 };

var source1 = { b: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
target // { a: 1, b: 2, c: 3 }
```

如果只有一个参数，会直接返回该参数；如果该参数不是对象，会先转换为对象，然后返回。由于`undefined`和`null`无法转换为对象，所以将它们作为参数传入会报错。

```javascript
var obj = { a: 1};
Object.assign(obj) === obj // true

typeof Object.assign(2) // object

Object.assign(undefined) // error
Object.assign(null) // error
```

如果`undefined`和`null`出现在非首参数，首先会尝试着转换为对象，如果转换失败会直接跳过。这意味着只要首参数可以转换为对象，就不会报错。

```javascript
let obj = { a: 1};

Object.assign(a, undefined) === obj // true
Object.assign(a, null) === obj // true
```

其他类型的值中，只有字符串会以数组的形式拷贝至对象中，其他值都不会产生效果；

```javascript
var v1 = 'abc';
var v2 = true;
var v3 = 10;

var obj = Object.assign({}, v1, v2, v3);
console.log(obj); // {'0': 'a', '1': 'b', '2': 'c'}
```

`Object.assign()`拷贝的属性是有限制的，只拷贝源对象的自身属性（不拷贝继承属性），也不拷贝不可枚举的属性（`enumerable: false`）。

```javascript
Object.assign({b: 'c'}, Object.defineProperty({}, 'invisible', {
    	enumerable: false,
    	value: 'hello'
	})
)
// {b: 'c'}
```

值得注意的是：`Object.assign()`执行的是浅拷贝，而不是深拷贝。如果源对象某个属性的值是对象，那么目标对象拷贝得到的仅仅是这个对象的引用。

```javascript
var obj1 = {a: {b: 1}};
var obj2 = Object.assign({}, obj1);

obj1.a.b = 2;
obj2.a.b // 2
```

### Object.assign()方法有哪些用途呢？

#### 为对象添加属性

```javascript
class Point {
    constructor(x, y) {
        Object.assign(this, {x, y})
    }
}
```

#### 为对象添加方法

```javascript
Object.assign(SomeClass.prototype, {
    someMethod(arg1, arg2) {
        ...
    },
    anotherMethod() {
        ...
    }
});
        
// 等同于下面的写法
SomeClass.prototype.someMethod = function (arg1, arg2) { ... };
SomeClass.prototype.anotherMethod = function () { ... };
```

#### 克隆对象

```javascript
// 将原始对象拷贝到空对象中
function clone (origin) {
    return Object.assign({}, origin);
}

// 将原始对象和其继承的值拷贝到新对象中
function clone (origin) {
    let originProto = Object.getPrototypeOf(origin);
    return Object.assign(Object.create(originProto), origin);
}
```

#### 合并多个对象

```javascript
// 将多个对象合并到某个对象
const merge = (target, ...source) => Object.assign(target, ...source);

// 合并后返回一个新对象
const merge = (...source) => Object.assign({}, ...source);
```

#### 为属性指定默认值

```javascript
const DEFAULTS = {
    logLevel: 0,
    outputFormat: 'html'
};

function processContent(options) {
    options = Object.assign({}, DEFAULTS, options)
}
```

## 属性的可枚举性

`Object.getOwnPropertyDescriptor(obj, 'foo')`方法可以获取该属性的描述对象，其中描述对象有一个`enumerable`属性，称为“可枚举性”，如果该属性为false，就表示某些操作会忽略当前属性。

ES6中有下面4个操作会忽略`enumerable`为false的属性。

- `for ... in` 循环：只遍历对象自身的和继承的可枚举属性；
- `Object.keys()`：返回对象自身的所有可枚举的属性键名；
- `JSON.stringify()`：只串行化对象自身的可枚举属性；
- `Object.assign()`：只拷贝对象自身的可枚举属性；

上面4个操作中，只有`for ... in`会返回继承的属性。实际上引入`enumerable`的最初目的，就是让某些可以规避掉`for...in`的操作。

```javascript
Object.getOwnPropertyDescriptor(Object.prototype, 'toString').enumerable // false

Object.getOwnPropertyDescriptor([], 'length').enumerable // false
```

另外ES6规定，所有Class的原型方法都是不可枚举的：

```javascript
Object.getOwnPropertyDescriptor(class {foo() {}}.prototype, 'foo').enumerable
```

## 属性的遍历

ES6中一共有5种方法可以遍历对象的属性

#### （1）for...in

遍历对象自身和继承的可枚举属性

#### （2）Object.keys(obj)

返回一个数组，包括对象自身的（不含继承）所有可枚举属性（不包括Symbol属性）

#### （3）Object.getOwnPropertyNames(obj)

返回一个数组，包含对象自身的所有属性（不包括Symbol属性）

#### （4）Object.getOwnPropertySymbols(obj)

返回一个数组，包含对象自身的所有Symbol属性

#### （5）Reflect.ownKeys(obj)

返回一个数组，包含对象自身的所有属性，不管是属性名还是Symbol还是字符串，也不管是否可枚举。

上面5种方法遍历对象属性，都遵循同样的属性遍历的次序规则。

- 首先遍历所有属性名为数值的字符的属性，按照数字排序；
- 其次遍历所有属性名为字符串的属性，按照生成时间排序；
- 最后遍历所有属性名为Symbol值得属性，按照生成时间排序；

一句话概括：遍历规则为数值 > 字符串 > Symbol；数值排序规则为数字，剩下两个排序规则为生成顺序；

## `__proto__`属性，`Objects.setPrototypeOf()`, `Object.getPrototypeOf()`

#### `__proto__`属性

用来读取或设置当前对象的`prototype`对象。目前所有浏览器都支持这个特性。

```javascript
var obj = {
    method: function () {...}
};
obj.__proto__ = someOtherObj;

var obj = Object.create(someOtherObj);
obj.method = function () {...};
```

这里和Python一样，使用双下划线标注的属性内部属性，而不是一个正式对外的API。因此在正式使用时不建议显示的设置。而是用`Object.setPrototypeOf()`、`Object.getPrototypeOf()`、`Object.create()`代替。

#### `Object.setPrototypeOf()`

用来设置对象的`prototype`对象。是ES6推荐的设置原型对象的方法。

```javascript
let proto = {};
let obj = { x: 10};
Object.setPrototypeOf(obj, proto);

proto.y = 20;
proto.z = 40;

console.log(obj.x) // 10
console.log(obj.y) // 20
console.log(obj.z) // 40
```

#### `Object.getPrototypeOf()`

与上面的set方法相反，用来获取一个对象的prototype对象。

```javascript
function Rectangle () {

}

var rec = new Rectangle();

Object.getPrototypeOf(rec) === Rectangle.prototype; // true

Object.setPrototypeOf(rec, Object.prototype);
Object.getPrototypeOf(rec) === Rectangle.prototype; // false
```

## `Object.values()`, `Object.entries()`

#### `Object.keys()`

返回一个数组，成员是参数对象自身的所有可遍历属性的键名（不含继承）

```javascript
var obj = { foo: "bar", baz: 42 };
Object.keys(obj)
// ["foo", "baz"]
```

目前ES7有一个新提案，引入了跟`Object.keys()`配套的`Object.values`和`Object.entries`。

```javascript
let { keys, values, entries } = Object;
let obj = { a: 1, b: 2, c: 3 };

for (let key  of keys(obj)) {
    console.log(key);
}
// 'a', 'b', 'c'

for (let value of values(obj)) {
    console.log(value);
}
// 1, 2, 3

for (let [key, value] of entries(obj)) {
    console.log([key, value]);
}
//['a', 1], ['b', 2], ['c', 3]
```

#### `Object.values()`

返回一个数组，成员是参数对象自身的（不含继承）所有可遍历（enumerable）属性的键值。

```javascript
var obj = { foo: "bar", baz: 42 };
Object.values(obj);
// ['baz', 42]
```

#### `Object.entries`

返回一个数组，成员是参数对象自身的（不含继承）所有可遍历属性的键值对数组。

```javascript
var obj = { foo: 'bar', baz: 42 };
Object.entries(obj);
// [ [ 'foo', 'bar' ], [ 'baz', 42 ] ]
```

基本用途为遍历对象的属性，也可以将对象转换为Map

```javascript
let obj = { one: 1, two: 2 };
for (let [k, v] of Object.entries(obj)) {
    console.log(`${JSON.stringify(k)} : ${JSON.stringify(v)}`)
}
// "one": 1
// "two": 2

// 另一个用途是将对象转换为Map
var obj = { foo: 'bar', baz: 42 };
var map = new Map(Object.entries(obj));
map // Map {foo: "bar", baz: 42 }
```

## 对象的扩展运算符

之前提到过扩展运算符（`...`），在对象中也有运用。

#### 解构赋值

对象的解构赋值用于从一个对象取值，相当于将所有可遍历的、但尚未被读取的属性分配到指定的对象上。所有的键和值，都会拷贝到新对象上。

```javascript
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x // 1
y // 2
z // { a: 3, b: 4 }
```

解构必须是最后一个参数，否则会报错，这和Java是完全一样的。注意一点：解构赋值时浅拷贝，如果一个键的值是复合类型的值（数组、对象、函数），那么解构赋值拷贝的是这个值的引用，而不是这个值的副本。

```javascript
let obj = { a: { b: 1 }};
let {...x} = obj;
obj.a.b = 2;
x.a.b // 2 (浅拷贝无疑了)
```

另外解构赋值不会拷贝继承自原型对象的属性。下面代码中，对象o3是o2的拷贝，但是只复制了o2自身的属性，没有复制它的原型对象o1的属性。

解构赋值的一个用处，是扩展某个函数的参数，引入其他操作：

```javascript
function baseFunction ({a, b}) {
    // ...
}

function wrapperFunction ({x, y, ...restConfig}) {
    return baseFunction(restConfig);
}
```

扩展运算符（`...`）用于取出参数对象的所有可遍历属性，拷贝到当前对象之中。

```javascript
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }
```

还可以用于合并两个对象。

```javascript
let ab = { ...a, ...b };
// 等同于
let ab = Object.assign({}, a, b);
```

## `Object.getOwnPropertyDescriptors()`

前面提到了：这个方法是用来返回某个对象属性的描述对象的。ES7有个提案：返回指定对象所有自身属性（非继承性——的描述对象。

```javascript
const obj = {
    foo: 123,
    get bar() { return 'abc' }
};

Object.getOwnPropertyDescription(obj)

// 返回结果如下
{
  foo: { value: 123, writable: true, enumerable: true, configurable: true },
  bar: {
    get: [Function: get bar],
    set: undefined,
    enumerable: true,
    configurable: true
  }
}
```

该方法实现的目的：主要是为了解决`Object.assing()`无法正确拷贝`get`属性和`set`属性的问题。结合`Object.defineProperties`方法就可以实现正确拷贝。

```javascript
const source = {
    set foo (value) {
        console.log(value);
    }
};

const target2 = {};
Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
Object.getOwnPropertyDescriptor(target2, 'foo')

// 返回结果
{                           
  get: undefined,           
  set: [Function: set foo], 
  enumerable: true,         
  configurable: true        
}                           
```

将上面两部分代码合并起来，就是这样。是不是突然觉得眼熟悉？（就是React Thunk的使用方法）

```javascript
const shallowMerge = (target, source) => Object.defineProperties(
	target,
    Object.getOwnPropertyDescriptors(source)
);

// 浅拷贝对象
const clone = Object.create(Object.getPrototypeOf(obj),
                           Object.getOwnPropertyDescriptors(obj));
const shallowClone = (obj) => Object.create(
	Object.getPrototypeOf(obj),
    Object.getOwnPropertyDescriptors(obj)
);
```
