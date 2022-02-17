---
title: ES6 Set和Map
date: 2022-02-17 15:16:56
updated: 2022-02-17 15:16:56
tags: ES6基础
categories: JavaScript
---

# Set和Map

## Set

### 基本用法

类似与数组结构，但是数组中的成员值时唯一不重复的。跟Java中的Set是同样的意思。

```javascript
var s = new Set();

[2, 3, 5, 4, 5, 2, 2].map(x => s.add(x));

for (let i of s) {
  console.log(i);
}
// 2 3 5 4
```

也可以结合扩展运算符写成下面这样：

```javascript
var set = new Set([1,2,3,4,5,2,3]);
[...set] // 1 2 3 4 5
set.size() // 5
```

### 实例的属性和方法

除了构造函数以及返回size的大小，还有下面4个操作的方法：使用基本和Java相同

- add(value)：添加某个值，返回Set结构本身
- delete(value)：删除某个值，返回一个布尔值，表示是否删除成功；
- has(value)：返回一个布尔值，表示该值是否为Set成员
- clear()：清除所有成员，没返回值。

```javascript
var s = new Set();
s.add(1).add(2).add(2);

s.size // 2

s.has(1) // true
s.has(2) // true
s.has(3) // false

s.delete(2);
s.has(2) // false
```

### 遍历操作

遍历器包含下面4个方法，可以用于遍历成员，结合Lambda表示可以有十分简洁的写法，跟Java 8的lambda表达式差不多，这里不再赘述。

- keys()：返回键名的遍历器
- values()：返回键值的遍历器
- entries()：返回键值对的遍历器
- forEach()：使用回调函数遍历每个成员

```javascript
let set = new Set(['red', 'green', 'blue']);
for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries) {
  console.log(item)
}
// ['red', 'red']
// ['green', 'green']
// ['blue', 'blue']
```

### WeakSet

从名字上看，WeakSet也是Set的一种，但是跟Set有下列几个区别：

- 成员只能是对象，不能是其他类型的值；
- 集合内的对象全都是弱引用，GC不会考虑对WeakSet中的对象的引用，因此WeakSet是不可遍历的。

你可能会问，WeakSet的这种特性有什么用呢？由于不能遍历且内存中的对象随时会丢失，因此十分适合存储DOM节点，而不用担心这些节点从文档移除时引发内存泄露。

```javascript
const foos = new WeakSet();

class foo {
  constructor() {
    foos.add(this);
  }
  
  method() {
    if (!foos.has(this)) {
      throw new TypeError('Foo.prototype.method智能在Foo实例上调用!')
    }
  }
}
```



## Map

### 基本用法

JavaScript的对象本质上也是键值对的集合，但是传统上只能用字符串当键。因为ES6提供了Map这个对象。概念跟Java中的差不多。与JavaScript中对象唯一不同的是：它的键不仅限于字符串，还可以是其他类型。

```javascript
var m = new Map();
var o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o)

m.has(o)
m.delete(o)
m.has(o)
```

与此同时，Map也可以接受一个数组作为参数。该数组的成员是一个个表示键值对的数组。

```javascript
var map = new Map([
    ['name', 'zhangsan'],
    ['title', 'Author']
]);

map.size // 2
map.has('name') // true
map.get('name') // zhangsan
map.has('title') // true
map.get('title') // Author
```

### 实例的属性和方法

跟上面提到的Set一样，Map也有属于自己的实例属性和方法。

- size：返回Map结构的成员总数
- set(key, value)：设置key对应的键值，然后返回整个Map结构。如果key已经有值，则键值会被更新；
- get(key)：读取key对应的键值，如果找不到则返回undefined；
- has(key)：返回一个布尔值，表示某个键是否在Map数据结构中；
- delete(key)：删除某个键，返回true，如果删除失败则返回false；
- clear()：清除所有成员，没有返回值；

### 遍历方法

Map提供了三个原生的遍历器生成函数和一个遍历方法，这个和之前的Set很相似；

- keys()：返回键名的遍历器
- values()：返回键值的遍历器
- entries()：返回所有成员的遍历器
- forEach()：返回Map的所有成员

下面提供一个实例的使用方法：

```javascript
let map = new Map([
    ['F', 'no'],
    ['T', 'yes']
]);

for (let key of map.keys()) {
    console.log(key);
}

for (let value of map.values()) {
    console.log(value);
}

for (let item of map.entries()) {
    console.log(item)
}

for (let [key, value] of map.entries()) {
    console.log(key, value)
}

for (let [key, value] of map) {
    console.log(key, value)
}
```

同样的，使用扩展运算符可以将Map快速转换为数组。

```javascript
let map = new Map([
    [1, 'one'],
    [2, 'two'],
    [3, 'three']
]);

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
// [[1, 'one'], [2, 'two'], [3, 'three']]

[...map]
// [[1, 'one'], [2, 'two'], [3, 'three']]
```

结合map和Filter等方法可以实现更加方便简洁的操作。

```javascript
let map0 = new Map()
	.set(1, 'a')
	.set(2, 'b')
	.set(3, 'c');

let map1 = new Map(
	[...map0].filter([k, v] => k < 3)
);

let map2 = new Map(
	[...map0].map([k, v] => [k * 2, '_' + v])
);
```

此外，还有forEach方法，可以实现遍历。效果和Set上的forEach完全一样

```javascript
map.forEach(function (key, value, map) {
    console.log("key: %s, value: %s", key, value);
});

// 还可以绑定this
map.forEach(function (key, value, map){
    this.report(key, value);
}, reporter);
```

### 与其他数据结构的互相转换

### Map转数组

前面已经提到，使用扩展运算符可以很好地将Map转换成数组，这里不再赘述；

### 数组转Map

将数组作为参数传入Map的构造函数，就可以将数组转换为Map

### Map转对象

如果Map中的所有键都是字符串，那么就可以将其转换为对象

```javascript
function strMapToObj(strMap) {
    let obj = Object.create(null);
    for (let [k, v] of strMap) {
        obj[k] = v;
    }
    return obj;
}

let myMap = new Map().set('yes', true).set('no', false);
strMapToObj(myMap)
```

### 对象转Map

```javascript
function objToStrMap(obj) {
    let strMap = new Map();
    for (let k of Object.keys(obj)) {
        strMap.set(k, obj[k]);
    }
    return strMap;
}

objToStrMap({yes: true, no: false})
```

### Map转JSON

Map转JSON要分为两种情况：如果键名都是字符串，那么可以直接转成JSON；如果键名中有非字符串，这时可以选择转为数组JSON。

```javascript
// Map --> JSON (1)
function strMapToJson(strMap) {
    return JSON.stringify(strMapToObj(strMap));
}

let myMap = new Map().set('yes', true).set('no', false);
strMapToJson(myMap)

function mapToArrayJson(map) {
    return JSON.stringify([...map]);
}
let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
mapToArrayJson(myMap)
```

### JSON转Map

跟上面的情况一样：所有键名都是字符串，随便转；另外一个：整个JSON就是数组，每个数组成员本身又只有一个两个成员的数组。这个时候可以一一对应转换为Map。

```javascript
function jsonToMap(jsonStr) {
    return new Map(JSON.parse(jsonStr));
}

jsonToMap('[[true, 7], [{"foo":3}, ["abc"]]]')
```

### WeakMap

WeakMap和Map基本类似，唯一区别就是：它只接受对象作为键名，不接受其他类型的值作为键名，而且键名所指向的对象不计入垃圾回收机制。跟WeakSet一样，设计它的目的有助于防止内存泄露：

```javascript
let myElement = document.getElementById('logo');
let myWeakMap = new WeakMap();

myWeakMap.set(myElement, {timesClicked: 0});

myElement.addEventListener('click', function(){
    let logoData = myWeakMap.get(myElement);
    logoData.timesClicked ++;
}, false);
```

上述代码设计了一个机制：每当DOM中节点发生click事件，就更新一下状态。一旦这个DOM节点删除，该状态就会自动消失，不会存在内存泄露的风险。

WeakMap的另一个作用就是部署私有属性

```javascript
let _counter = new WeakMap();
let _action = new WeakMap();

class Countdown {
    constructor(counter, action) {
        _counter.set(this, counter);
        _action.set(this, action);
    }
    dec() {
        let counter = _counter.get(this);
        if (counter < 1) return;
        counter--;
        _counter.set(this, counter);
        if (counter === 0) {
            _action.get(this)();
        }
    }
}

let c = new Countdown(2, () => console.log('DONE'));

c.dec()
c.dec()
// DONE
```

上面代码中，Countdown类的两个内部属性`_counter`和`_action`是实例的弱引用，所以如果删除实例，它们也就随之消失。不会造成内存泄露。
