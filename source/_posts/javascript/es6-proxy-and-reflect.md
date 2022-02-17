---
title: ES6 Proxy和Reflect
date: 2022-02-17 15:18:01
updated: 2022-02-17 15:18:01
tags: ES6基础
categories: JavaScript
---

# Proxy和Reflect

## Proxy

### 概述

Proxy用于修改某些操作的默认行为，等同于在语言层面做修改，所以属于一种“元编程”。Proxy可以理解为在目标对象之前架设一层拦截，外界对该对象的访问必须先通过这层拦截，因此提供了一种机制：可以对外界的访问进行过滤和改写。例子如下：

```javascript
var obj = new Proxy({}, {
    get: function (target, key, receiver) {
        console.log(`getting ${key}!`);
        return Reflect.get(target, key, receiver);
    },
    set: function (target, key, value, receiver) {
        console.log(`setting ${key}`);
        return Reflect.set(target, key, value, receiver);
    }
});
```

ES6原生提供了Proxy构造函数，用来生成Proxy实例。

```javascript
var proxy = new Proxy(target, handler);
```

其中target表示所要拦截的对象，handler用来定制拦截行为。例子如下：

```javascript
var proxy = new Proxy({}, {
    get: function(target, property) {
        return 35;
    }
});

proxy.time // 35
proxy.name // 35
proxy.titile // 35
```

需要说明的是：如果handler没有设置任何拦截对象，那么将继续保持原对象的行为不变。并且同一个拦截函数可以设置多个拦截操作：

```javascript
var handler = {
    get: function(target, name) {
        if (name === 'prototype') {
            return Object.prototype;
        }
        return 'Hello, ' + name;
    },

    apply: function(target, thisBinding, args) {
        return args[0];
    },
    construct: function(target, args) {
        return {value: args[1]};
    }
};

var fproxy = new Proxy(function (x, y){
    return x + y;
}, handler);

console.log(fproxy(1, 2)); // 1
console.log(new fproxy(1, 2)); // {value: 2}
console.log(fproxy.prototype === Object.prototype); // true
console.log(fproxy.foo) // "Hello, foo"
```

下面是Proxy支持的拦截操作一览：

- get(target, propKey, receiver)：拦截对象属性的读取
- set(target, propKey, value, receiver)：拦截对象属性的设置
- has(target, propKey)：拦截`propKey in proxy`的操作，以及对象的`hasOwnProperty`方法，返回一个布尔值
- deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值；
- ownKeys(target)：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`返回一个数组。该方法返回对象所有自身的属性
- getOwnPropertyDescriptor(target, propKey)：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`返回属性的描述对象。
- defineProperty(target, propKey, propDesc)：拦截`Object.defineProperty(proxy, propKey, propDesc)`返回一个布尔值；
- preventExtensions(target)：拦截`Object.preventExtensions(proxy)`，返回一个布尔值；
- getPrototypeOf(target)：拦截`Object.getPropertyOf(proxy)`返回一个对象
- isExtensible(target)：拦截`Object.isExtensible(target)`，返回一个布尔值
- setPrototypeOf(target, proto)：拦截`Object.setPrototypeOf(proxy, proto)`返回一个布尔值；
- apply(target, object, args)：拦截Proxy实例作为函数的调用操作
- construct(target, args)：拦截Proxy实例作为构造函数调用的操作

### 实例方法

#### get()

```javascript
var person = {
    name: "zhangsan"
};

var proxy = new Proxy(person, {
    get: function(target, property) {
        if (property in target) {
            return target[property];
        } else {
            throw new ReferenceError("Property \"" + property + "\" doesn't exist.");
        }
    }
});
```

#### set()

```javascript
let validator = {
    set: function(target, prop, value) {
        if (prop === 'age') {
            if (!Number.isInteger(value)) {
                throw new TypeError('The age is not an integer');
            }
            if (value > 200) {
                throw new RangeError('The age seems invalid');
            }
        }

        target[prop] = value;
    }
};

let person = new Proxy({}, validator);
person.age = 100;

console.log(person.age);
person.age = 'young';
console.log(person.age);
```

#### apply()

```javascript
var target = function () { return "I am the target;"; };
var handler = {
    apply: function() {
        return "I am the proxy";
    }
};

var p = new Proxy(target, handler);

console.log(p()); // I am the proxy
```

#### has()

```javascript
let stu1 = { name: 'zhangsan', score: 59 };
let stu2 = { name: 'lisi', score: 99 };

let handler = {
    has(target, prop) {
        if (prop === 'score' && target[prop] < 60) {
            console.log(`${target.name} 不及格`);
            return false;
        }
        return prop in target;
    }
}

let oproxy1 = new Proxy(stu1, handler);
let oproxy2 = new Proxy(stu2, handler);

console.log('score' in oproxy1)
console.log('score' in oproxy2)

for (let a in oproxy1) {
    console.log(oproxy1[a]);
}

for (let b in oproxy2) {
    console.log(oproxy2[b]);
}
```

#### construct()

```javascript
var p = new Proxy(function() {}, {
    construct: function(target, args) {
        console.log('called: ' + args.join(', '));
        return { value: args[0] * 10 };
    }
});

console.log(new p(1).value);
```

#### deleteProperty()

```javascript
var handler = {
    deleteProperty(target, key) {
        invariant(key, 'delete');
        return true;
    }
};

function invariant(key, action) {
    if (key[0] === '_') {
        throw new Error(`Invalid attempt to ${action} private "${key}" property`);
    }
}

var target = { _prop: 'foo' };
var proxy = new Proxy(target, handler);
delete proxy._prop
```

#### defineProperty()

```javascript
var handler = {
    defineProperty(target, key, descriptor) {
        return false;
    }
};

var target = {};
var proxy = new Proxy(target, handler);
proxy.foo = 'bar';
```

#### getOwnPropertyDescriptor()

```javascript
var handler = {
  getOwnPropertyDescriptor (target, key) {
    if (key[0] === '_') {
      return;
    }
    return Object.getOwnPropertyDescriptor(target, key);
  }
};
var target = { _foo: 'bar', baz: 'tar' };
var proxy = new Proxy(target, handler);
Object.getOwnPropertyDescriptor(proxy, 'wat')
// undefined
Object.getOwnPropertyDescriptor(proxy, '_foo')
// undefined
Object.getOwnPropertyDescriptor(proxy, 'baz')
// { value: 'tar', writable: true, enumerable: true, configurable: true }
```

#### getPrototypeOf()

```javascript
var proto = {};
var p = new Proxy({}, {
    getPrototypeOf(target) {
        return proto;
    }
});

Object.getPrototypeOf(p) === proto // true
```

#### isExtensible()

```javascript
var p = new Proxy({}, {
    isExtensible: function(target) {
        console.log('called');
        return false;
    }
});

Object.isExtensible(p)
```

#### ownKeys()

```javascript
let target = {
    _bar: 'foo',
    _prop: 'bar',
    prop: 'baz'
};

let handler = {
    ownKeys(target) {
        return Reflect.ownKeys(target).filter(key => key[0] !== '_');
    }
};

let proxy = new Proxy(target, handler);
for (let key of Object.keys(proxy)) {
    console.log(target[key]); // baz
}
```

### 取消代理

Proxy.revocable()方法返回一个可取消的Proxy实例。

```javascript
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```

### this问题

虽然Proxy可以代理针对目标对象的访问，但不是目标对象的透明代理：即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因是在Proxy代理的情况下，目标对象内部的`this`关键字会指向Proxy代理。

```javascript
const _name = new WeakMap();

class Person {
    constructor(name) {
        _name.set(this, name);
    }

    get name() {
        return _name.get(this);
    }
}

const jane = new Person('Jane');
jane.name; // "Jane"

const proxy = new Proxy(jane, {});
proxy.name // undefined
```

## Reflect

### 概述

Reflect也是ES6为了操作对象而提供的新API，其设计目的主要有几个：

1. 将Object对象的一些明显属于语言内部的方法放到Reflect对象上。
2. 修改某些Object方法返回的结果，让其变得合理。
3. 让Object的操作都变成函数行为。某些Object操作是命令式的；
4. Reflect对象的方法与Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。

```javascript
var loggedObj = new Proxy(obj, {
    get(target, name) {
        console.log('get', target, name);
        return Reflect.get(target, name);
    },
    deleteProperty(target, name) {
        console.log('delete' + name);
        return Reflect.deleteProperty(target, name);
    },
    has(target, name) {
        console.log('has' + name);
        return Reflect.has(target, name);
    }
});
```

### Reflect对象的方法

大体上和Object对象的同名方法的作用是相同的，而且与Proxy对象的方法是一一对应的。
