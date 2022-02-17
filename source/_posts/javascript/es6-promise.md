---
title: ES6 Promise对象
date: 2022-02-17 22:20:53
updated: 2022-02-17 22:20:53
tags: ES6基础
categories: JavaScript
---

# Promise对象

## 含义

Promise是异步编程的一种解决方案，最早由社区提出和实现，ES6在次基础上统一了用法并且提供了Promise对象。

所谓的Promise简单来说就是一个容器：里面保存着某个未来才会结束的事件的结果。从语法上说：Promise是一个对象，从它可以获取异步操作的消息。

Promise对象由两个特点：

- 对象的状态不受外界影响，只有异步操作的结果可以决定当前是哪一种状态，任何其他操作都无法改变这种状态；
- 一旦状态发生了改变，就不会再变，任何时候都可以得到这个结果。

因此：有了Promise，就可以将异步操作以同步的流程表达出来，避免了蹭蹭嵌套的回调函数。此外，Promise还提供了统一的接口，使得控制异步操作更加容易。

但是Promise也存在缺点：例如一旦创建就会立即执行，无法中途取消，其次，如果不设置回调函数，Promise内部会抛出异常，不会反应到外部。最后，当处于Pending状态的时候，无法得知目前进展到哪一个阶段。

## 基本用法

ES6规定，Promise对象是一个构造函数，用来生成Promise实例。

```javascript
var promise = new Promise(function(resolve, reject){
    if (a) {
        resolve(value); // Pending -> Resolved
    } else {
        reject(error); // Pending -> Rejected
    }
});

promise.then(function(value){
    // success
}, function(error){
   // failure 
});
```

Promise新建后会立即执行

```javascript
let promise = new Promise(function(resolve, reject){
    console.log('Promise');
    resolve();
});

promise.then(function() {
    console.log('Resolved.')
});

console.log('Hi!');

// Promise
// Hi!
// Resolved
```

## Promise.prototype.then()

`then`方法是定义在原型对象Promise.prototype上的。它的作用是为Promise实例添加状态改变时的回调函数。该方法返回的是一个新的Promise实例。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法。

采用链式的`then`，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。

```javascript
getJSON("/posts.json").then(function(json){
    return json.post;
}).then(function(post){
    // ...
});
```

如果使用箭头函数，可以将上面代码进一步简化。

```javascript
getJSON("/post/1.json").then(
	post => getJSON(post.commentURL)
).then(
	comments => console.log("Resolved: " + comments),
    err => console.log("Rejected: ", err)
);
```

## Promise.prototype.catch()

用于指定发生错误时的回调函数。

```javascript
getJSON("/post.json").then(function(posts){
    // ...
}).catch(function(error){
    console.log('Error ', error);
});
```

如果Promise状态已经变成了Resolved，再抛出错误就是无效的。

```javascript
var promise = new Promise(function (resolve, reject){
    resolve('ok');
    throw new Error('test');
});

promise
	.then(function(value) {console.log(value)})
	.catch(function(value) {console.log(error)});
// ok
```

Promise对象错误具有”冒泡“性质，会一直向后传递，直到被捕获为止。也就是说：错误总会被下一个`catch`语句捕获。

```javascript
getJSON("/post/1.json").then(function(post){
    return getJSON(post.commentURL);
}).then(function(comments){
    // some code
}).catch(function(error){
    // error handler
});
```

与传统的`try/catch`代码块不同的是，如果没有使用`catch`方法指定错误处理的回调函数，Promise对象抛出的错误不会传递到外层代码。但是Chrome不遵守这个规则，还是会抛出异常。好在Node.js提供了一个`unhandledRejection`事件，专门监听未捕获的`reject`错误。

```javascript
process.on('unhandledRejection', function(err, p){
    console.error(err.stack);
});

var someAsyncThing = function() {
    return new Promise(function(resolve, reject){
        resolve(x + 2);
    });
};

someAsyncThing().then(function() {
    console.log('everything is great');
});
```

`catch`方法中还可以继续抛出异常。例如下面的代码：第二个`catch`方法用来捕获，前一个`catch`方法抛出错误。

```javascript
someAsyncThing().then(function() {
  return someOtherAsyncThing();
}).catch(function(error) {
  console.log('oh no', error);
  // 下面一行会报错，因为y没有声明
  y + 2;
}).catch(function(error) {
  console.log('carry on', error);
});
// oh no [ReferenceError: x is not defined]
// carry on [ReferenceError: y is not defined]
```

## Promise.all()

`Promise.all`方法用于将多个Promise实例，包装成一个新的Promise实例。语法如下：

```javascript
var p = Promise.all([p1, p2, p3]);
```

例子如下：只有这6个实例的状态都变成了`fulfilled`，或者其中有一个状态变成了`rejected`，才会调用`Promise.all`方法后面的回调函数。

```javascript
var promises = [2, 3, 5, 7, 11, 13].map(function (id){
    return getJSON("/post/" + id + ".json");
});

Promise.all(promises).then(function (post){
    // ...
}).catch(function(reason){
    // ...
});
```

再例如下面这个例子：只有`booksPromise`和`userPromise`的结果都返回了，才会触发`pickTopRecommentations`这个回调函数。

```javascript
const databasePromise = connectDatabase();

const booksPromise = databasePromise.then(findAllBooks);

const userPromise = databasrPromise.then(getCurrentUser);

Promise.all([
    booksPromise,
    userPromise
]).then([books, user] => pickTopRecommentations(books, user));
```

## Promise.race()

跟上面的作用一样，将多个Promise实例合成一个新的Promise实例。

```
var p = Promise.race([p1, p2, p3]);
```

上面的例子中：只要三个状态有其中一个率先改变，新对象的状态就跟着一起改变。率先改变的Promise实例的返回值就是新Promise实例的回调函数的输入。

下面提供一个实例：如果指定时间内没有获得结果，就将Promise的状态变为`reject`，否则就为`resolve`

```javascript
var p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
])
p.then(response => console.log(response))
p.catch(error => console.log(error))
```

## Promise.resolve()

有时需要将现有对象转为Promise对象，Promise.resolve()方法就起这个作用。

```javascript
Promise.resolve('foo')
// equal to
new Promisr(resolve => resolve('foo'));
```

`Promise.resolve`方法的参数可以分为4种情况。

- 参数是一个Promise实例：不做任何修改，原封不动返回实例
- 参数是一个thenable对象：将该对象转换为Promise对象，然后立即执行`thenable`对象的`then`方法；
- 参数不是具有then方法的对象，或根本就不是对象：返回一个新的`Promise`对象，状态为`Resolved`；
- 不带有任何参数：直接返回一个状态为`Resolved`的Promise对象；

## Promise.reject()

也会返回一个新的Promise实例，该实例的状态为`rejected`。其参数用法和`Promise.resolve()`方法完全一致；

## 两个有用的附加方法

### done()

提供一个`done`方法，总是处于回调链的尾端，保证抛出任何可能出现的错误

### finally()

用于指定不管Promise对象最后状态如何，都会执行的操作。与`Done`方法最大的区别，接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。

## 应用

### 加载图片

我们可以将图片加载写成一个Promise，一旦加载完成，`Promise`的状态就发生变化；

### Generator函数结合Promise

使用Generator函数管理函数流程，遇到异步操作的时候，通常返回一个`Promise`对象；

### Promise.try()

实际开发中还遇到一种情况：不知道或者不想区分函数f是同步还是异步，但还是想用Promise来处理。

实际开发中提供了两种写法来实现这种效果：

```javascript
// async 写法
const f = () => console.log('now');
(async () => f())();
console.log('next');

// Promise写法
const f = () => console.log('now');
{
    () => new Promise(
    	resolve => resolve(f())
    )
}();
console.log('next');
```

需要注意的是`async () => f()`会吃掉`f()`抛出的异常，因此需要使用`promise.catch`方法来捕获异常；
