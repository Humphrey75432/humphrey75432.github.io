---
title: ES6 Generator函数
date: 2022-02-17 22:20:37
updated: 2022-02-17 22:20:37
tags: ES6基础
categories: JavaScript
---

# Generator函数

## 基本概念

Generator函数是ES6提供的一种异步编程解决方案，语法行为与传统函数完全不同。

Generator函数有多种理解角度：从语法上：可以将其理解成状态机，封装了多个内部状态；其次：执行Generator函数会返回一个遍历器对象。

形式上，Generator函数是个普通函数，但是有两个特征。一是`function`关键字和函数名之间多了一个星号；二是函数体内部使用了`yield`语句，定义了不同的内部状态。

```javascript
function* hello() {
    yield 'hello';
    yield 'world';
    return 'ending';
}

var hello = hello();
hello.next() // hello - 1
hello.next() // world - 2
hello.next() // ending - 3
hello.next() //undefined - 4
```

## yield语句

可以看出，只有显式调用next()方法才会继续遍历到下一个状态，所以其实提供了一种可以暂停执行的函数。其中yield就是暂停标识。如果Generator函数不使用yield语句，这时就变成了一个单纯地暂缓执行函数。注意：yield不能用在普通函数中。

```javascript
function* f() {
    console.log('Executed!');
}

var generator = f();
setTimeout(function() {
    generator.next();
}, 2000);
```

## 与Iterator接口的关系

由于Generator函数就是遍历器生成的函数，因此可以把Generator赋值给对象的Symbol.iterator属性，从而使得该对象具有Iterator接口。

```javascript
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
    yield 1;
    yield 2;
    yield 3;
};
[...myIterable] // [1, 2, 3]
```

## next()方法的参数

`yield`语句本身没有返回值，或者说总是返回`undefined`。`next`方法可以带一个参数，该参数就会被当做上一个yield语句的返回值。

```javascript
function* f() {
    for (var i = 0; true; i++) {
        var reset = yield i;
        if (reset) {i = -1;}
    }
}

var g = f();

g.next()
g.next()
g.next(true)
```

上面这个例子有一个很有意思的特性：可以通过在Generator函数运行的不同阶段，从外部向内部注入不同的值，从而来调整函数的行为。

## for...of循环

`for...of`循环可以自动遍历Generator函数生成的Iterator对象，且此时不再需要调用next方法。

```javascript
function* foo() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    yield 5;
    return 6;
}

for (let v of foo()) {
    console.log(v);
}
// 1 2 3 4 5
```

需要注意的是：一旦next方法的返回对象的done属性为true，for...of循环就会终止，且不包含该返回对象。下面再举一个解斐波那契数列的例子。

```javascript
function* fibonacci() {
    let [prev, curr] = [0, 1];
    for (;;) { // 内部自动会取下一个yield语句
        [prev, curr] = [curr, prev + curr];
        yield curr;
    }
}

for (let n of fibonacci()) {
    if (n > 1000) break;
    console.log(n);
}
```

利用`for...of`循环可以写出遍历任意对象的方法。原生的JavaScript对象没有遍历接口，无法使用`for...of`循环，通过Generator函数为它加上这个接口，就可以使用了。

```javascript
function* objectEntries() {
    let propKeys = Reflect.ownKeys(obj);
    
    for (let propKey of propKeys) {
        yield [propKey, obj[propKey]];
    }
}

let jane = { first: 'Jane', last: 'Doe' };

for (let [key, value] of objectEntries(jane)) {
    console.log(`${key}: ${value}`);
}

// 另一种写法是将Generator函数加入到Symbol.iterator属性中.
jane[Symbol.iterator] = objectEntries;
```

## Generator.prototype.throw()

Generator函数返回的遍历器对象，都有一个`throw`方法，可以在函数体外抛出错误，然后在Generator函数体内捕获。切记：不要混淆遍历器对象的throw方法和全局throw命令。因此：如果Generator函数内部没有部署`try...catch`代码块，那么`throw`方法抛出的错误，将被外部`try...catch`代码块捕获。

```javascript
var g = function* () {
    try {
        yield;
    } catch (e) {
        console.log('Inner catch', e);
    }
};

var i = g();
i.next();
i.throw(new Error('Error occurs!'));
```

如果Generator函数内部和外部都没有部署`try...catch`代码块，那么程序将直接报错并且中断运行。

```javascript
var gen = function* gen() {
    yield console.log('hello');
    yield console.log('world');
}

var g = gen();
g.next();
g.throw();
```

如果Generator函数内部部署了`try-...catch`代码块，就不会影响下一次yield语句的执行。throw方法在被捕获以后会顺带执行下一条yield语句。

```javascript
var gen = function* gen() {
    try {
        yield console.log('a');
    } catch (e) {
        // ...
    }
    yield console.log('b');
    yield console.log('c');
}

var g = gen();
g.next();
g.throw();
g.next();
```

## Generator.prototype.return()

Generator函数的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历Generator函数。

```javascript
function* gen() {
    yield 1;
    yield 2;
    yield 3;
}

var g = gen();

g.next(); // {value:1, done: false}
g.return('foo'); // {value: 'foo', done: true}
g.next(); // {value: undefined, done: true}
```

如果`return`方法调用时，不提供参数，则返回的`value`值为`undefined`;

```javascript
function* gen() {
    yield 1;
    yield 2;
    yield 3;
}

var g = gen();

g.next(); // {value: 1, done: false}
g.return(); // {value: undefined, done: true}
```

如果Generator函数内部有`try...finally`代码块，那么`return`方法将会推迟到`finally`代码块执行完再执行。

```javascript
function* numbers () {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}
var g = numbers()
g.next() // { done: false, value: 1 }
g.next() // { done: false, value: 2 }
g.return(7) // { done: false, value: 4 }
g.next() // { done: false, value: 5 }
g.next() // { done: true, value: 7 }
```

## yield* 语句

该语句是用来在一个Generator函数中执行另外一个Generator函数的。从语法的角度上看：如果yield语句后面跟着一个遍历器对象，需要在yield命令后面加上星号，表明它返回的是一个遍历器对象。

```javascript
function* bar() {
    yield 'x';
    yield* foo();
    yield 'y';
}

function* foo() {
    yield 'a';
    yield 'b';
}
```

## Generator函数的this

Generator函数总是返回一个遍历器，ES6规定这个遍历器是Generator函数的实例，也继承了Generator函数的`prototype`对象上的方法。

```javascript
function* g() {}

g.prototype.hello = function() {
    return 'hi!';
};

let obj = g();

obj instanceof g;
obj.hello(); // 'hi!'
```

默认情况下，Generator函数是不可以使用new关键字生成的，但是我们可以用下面这种方式来生成一个空的Generator实例。

```javascript
function* F() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}
var obj = {};
var f = F.call(F.prototype);

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

obj.a // 1
obj.b // 2
obj.c // 3

```

再将F改造成构造函数，就可以使用new命令来创建对象了；

```javascript
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

function F() {
  return gen.call(gen.prototype);
}

var f = new F();

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
```

## Generator函数的应用

### （1）异步操作的同步化表达

基于Generator函数暂停执行的效果，意味着可以把异步写到yield语句中。等到next方法时再往后执行。这实际上等同于不需要再写回调函数了，因为异步操作的后续可以放在yield语句的下面。具体例子前面已经有涉及到了，所以这里就不再演示了。

### （2）控制流管理

如果一个多步操作非常耗时，可以使用Generator函数进一步改善代码的运作流程。不过这种写法只适合同步操作。

```javascript
function* longRunningTask(value1) {
  try {
    var value2 = yield step1(value1);
    var value3 = yield step2(value2);
    var value4 = yield step3(value3);
    var value5 = yield step4(value4);
    // Do something with value4
  } catch (e) {
    // Handle any error from step1 through step4
  }
}

scheduler(longRunningTask(initialValue));

function scheduler(task) {
  var taskObj = task.next(task.value);
  // 如果Generator函数未结束，就继续调用
  if (!taskObj.done) {
    task.value = taskObj.value
    scheduler(task);
  }
}
```

### （3）部署Iterator接口

利用Generator函数的特性，可以在任意对象上部署Iterator接口。

```javascript
function* iterEntries(obj) {
  let keys = Object.keys(obj);
  for (let i=0; i < keys.length; i++) {
    let key = keys[i];
    yield [key, obj[key]];
  }
}

let myObj = { foo: 3, bar: 7 };

for (let [key, value] of iterEntries(myObj)) {
  console.log(key, value);
}

// foo 3
// bar 7
```

### （4）作为数据结构

Generator可以看作数据结构，更确切的说是一种数组结构，因为Generator函数可以返回一系列的值。这意味着它可以对任意表达式，提供类似数组的接口。

```javascript
function *doStuff() {
  yield fs.readFile.bind(null, 'hello.txt');
  yield fs.readFile.bind(null, 'world.txt');
  yield fs.readFile.bind(null, 'and-such.txt');
}
```
