---
title: ES6 异步操作
date: 2022-02-17 22:23:01
updated: 2022-02-17 22:23:01
tags: ES6基础
categories: JavaScript
---

# 异步操作和Async函数

## 基本概念

前面提到，JavaScript本身是不支持异步编程的，但是不支持异步处理，那么处理速度会特别慢。因此引入异步编程的特性便迫在眉睫。

按照前面章节提到的，JS在异步编程中实现的方式有下面几个：

- 回调函数
- Promise
- Generator函数 + 协程

## Thunk函数

简单来说：所谓的“船名调用”实现：就是将参数方法放到一个临时函数中，再将这个临时函数传入函数体，这个临时函数就称作“Thunk函数”：

```javascript
var thunk = function() {
  return x + 5;
}

function (thunk) {
  return thunk() * 2;
}
```

在JavaScript中函数替换的不是表达式，而是多参数函数，将其替换程单参数的版本。且只接受回调函数作为参数。因此，任何函数只要有回调函数，就可以写成Thunk函数的形式。

```javascript
var thunk = function(fn) {
	return function (...args) {
    return function(callback) {
      return fn.call(this, ...args, callback);
    }
  }
}
```

举一个具体的例子：

```javascript
function f(a, b) {
  cb(a);
}
let ft = Thunk(f);

let log = console.log.bind(console);
ft(1)(log) // 1
```

## Thunkify模块

生产环境下转换器，可以使用Thunkify模块。使用`npm install thunkify`安装；

```javascript
var thunkify = require('thunkify');
var fs = require('fs');

var read = thunkify(rs.readFile);
read('package.json')(function(err, str){
  // concrete code
});
```

## 看到这你可能会问，这有什么用？

因为ES6现在新增了Generator函数，因此结合Thunk以后可以很好地管理Generartor流程。例子如下：

```javascript
var fs = require('fs');
var thunkify = require('thunkify');
var readFile = thunkify(fs.readFile);

var gen = function* () {
  var r1 = yield readFile('/etc/fstab');
  console.log(r1.toString());
  var r2 = yield readFile('/etc/shells');
  console.log(r2.toString());
}
```

上面的代码：yield命令用于将程序的执行权移出Generator函数。Thunk函数正好可以完成这个需求。

## 还没结束

看完上面，你可能已经知道Thunk函数现在可以做到什么事情：没错，Thunk函数的真正威力是：可以自动执行Generator函数。

```javascript
function run(fn) {
  var gen = fn();
  
  function next(err, data) {
    var result = gen.next(data);
    if (result.done) return;
    result.value(next);
  }
  
  next();
}

function* g() {
  // ...
}

run(g);
```

有了上面的执行器，那么下面执行Generator函数就简单了很多。不关内部有多少个异步操作，直接把Generator函数传入run函数即可。前提是每一个异步操作都必须是Thunk函数。

```javascript
var g = function* () {
  var f1 = yield readFile('fileA');
  var f2 = yield readFile('fileB');
  // ...
  var fn = yield readFile('fileN');
}

run(g);
```

## CO模块

co模块是著名程序员TJ Holowaychuk发布一个小工具，用于Generator函数的自动执行。有了co模块，你可以不必自己编写Generator函数的执行器。

```javascript
var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

var co = require('co');
co(gen).then(function() {
  console.log('Generator execute success.')
});
```

co支持并发的异步操作，即允许某些操作同时进行，等到它们全部完成，才进行下一步。

```javascript
// 数组写法
co(function* (){
  var res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ];
  console.log(res);
}).catch(onerror);

// 对象写法
co(function* (){
  var res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2)
  };
  console.log(res);
}).catch(onerror);
```

## async函数

ES7提供了async函数，使得异步操作变得更加方便。async函数是什么？一句话：async函数就是Generator函数的语法糖。

```javascript
var asyncReadFile = async function() {
  var f1 = await readFile('/etc/fstab');
  var f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

语法上来看，跟Generator函数的使用基本类似，但是在其基础上还是有一点改进：

- （1）内置执行器：因此async函数执行和普通函数一样，只需要一行即可；
- （2）更好的语义，这个一目了然；
- （3）更广的适用性：即可支持Promise，也可以支持普通对象；
- （4）返回值是Promise，因此可以使用then指定下一步操作；

如果async函数内部抛出错误，会导致返回的Promise对象变为reject状态。抛出的对象会被catch方法回调接收到。

```javascript
async function f() {
  throw new Error('Wrong');
}

f().then(
	v => console.log(v),
  e => console.log(e)
)
```

其次：async函数返回的Promise对象，必须等到所有的await命令的Promise对象执行完，才发发生改变。

```javascript
async function getTitle(url) {
	let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}

getTitle('https://github.com/Humphrey75432').then(console.log);
```

### 注意点

（1）`await`命令后面的`Promise`对象，运行结果可能是`reject`，所以最好把`await`命令放入`try...catch`块中。

```javascript
async function myFunction() {
    try {
        await somethingThatReturnsAPromise();
    } catch (err) {
        console.log(err);
    }
}

// Another grammar

async function myFunction() {
    await somethingThatReturnsAPromise()
    	.catch(function (err) {
        console.log(err);
    });
}
```

（2）多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。这样会提升执行效率

```javascript
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// second grammar
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

（3）如果确实希望多个请求并发执行，可以使用`Promise.all`方法。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 或者使用下面的写法

async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}
```
