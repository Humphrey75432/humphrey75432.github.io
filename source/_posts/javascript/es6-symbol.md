---
title: ES6 Symbol
date: 2022-02-17 15:15:26
updated: 2022-02-17 15:15:26
tags: ES6基础
categories: JavaScript
---

# Symbol

ES5的对象属性名都是字符串，这容易造成属性名的冲突。如果存在一种机制，能够保证每个属性的名字都是独一无二的就可以解决这个问题。因此ES6中引入了Symbol。

ES6引入了一种新的原始数据类型Symbol，表示独一无二的值。可以称之为JavaScript的第七种数据类型。

Symbol是通过symbol函数生成的。也就是说，对象的属性名可以有两种类型，一种是原来就有的字符串，另一种是新增的Symbol类型。凡是属性名属于Symbol的，就是独一无二的，可以保证不会与其他属性名发生冲突。

```javascript
let s = Symbol();
typeof s // symbol
```

`Symbol`函数可以接收一个字符串作为参数，表示对Symbol实例的描述，主要是为了在控制台显示或者是转为字符串的过程中方便区分。

```javascript
var s1 = Symbol('foo');
var s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"
```

如果Symbol的参数是一个对象，那么就会调用该对象的`toString()`方法，将其转换为字符串，然后才生成一个Symbol值。

```javascript
const obj = {
    toString() {
        return 'abc';
    }
};

const sym = Symbol(obj);
sym // Symbol(abc)
```

Symbol值不能与其他类型的值进行运算，会报错；但是却可以显式转换为字符串

```javascript
var sym = Symbol('My Symbol');

"your symbol is " + sym // Error

String(sym); // Symbol('My symbol')
sym.toString(); // Symbol(My symbol)
```

## 作为属性名的Symbol

由于每一个Symbol值都是不相等的，这意味着Symbol可以作为标识符，用在对象的属性名，就能保证不会出现同名的属性。可以防止某一个键被不小心改写或覆盖。

```javascript
var mySymbol = Symbol();

var a = {};
a[mySymbol] = 'Hello!';

var a = {
    [mySymbol]: 'Hello'
};

var a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello' });

a[mySymbol] // "Hello"
```

Symbol值作为对象属性名时，不能用作点运算符。

```javascript
var mySymbol = Symbol();
var a = {};

a.mySymbol = 'Hello';
a[mySymbol] // undefined
a['mySymbol'] // Hello
```

Symbol类型还可以用于定义一组常量，保证这组常量的值都是不相等的。

```javascript
log.levels = {
    DEBUG: Symbol('debug'),
    INFO: Symbol('info'),
    WARN: Symbol('warn')
};

log(log.levels.DEBUG, 'debug message');
log(log.levels.INFO, 'info message');
```

还有一个例子：

```javascript
var shapeType = {
    triangle: Symbol()
};

function getArea(shape, options) {
    var area = 0;

    switch(shape) {
        case shapeType.triangle:
            area = .5 * options.width * options.height;
            break;
    }

    return area;
}

getArea(shapeType.triangle, {width: 100, height: 100}); // 5000
```

## 属性名的遍历

Symbol作为属性名，不会出现在`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`返回。但是，它也不是私有属性，有一个`Object.getOwnPropertySymbols`方法，可以获取指定对象的所有Symbol属性名。

```javascript
var obj = {};
var a = Symbol('a');
var b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';

var objectSymbols = Object.getOwnPropertySymbols(obj);

objSymbols // [Symbol(a), Symbol(b)]
```

由于以Symbol值作为名称的属性，不会被常规方法遍历得到。因此可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法。

```javascript
var size = Symbol('size');

class Collection {
    constructor() {
        this[size] = 0;
    }
    
    add(item) {
        this[this[size]] = item;
        this[size]++;
    }
    
    static sizeOf(instance) {
        return instance[size];
    }
}

var x = new Collection();
Collection.sizeOf(x) // 0

x.add('foo');
Collection.sizeOf(x) // 1

Object.keys(x) // ['0']
Object.getOwnPropertyNames(x) // ['0']
Object.getOwnPropertySymbols(x) // [Symbol(size)]
```

## Symbol.for(), Symbol.keyFor()

有时候希望重新使用同一个Symbol值，`Symbol.for`可以做到这一点。其接受一个字符串为参数，然后搜索有没有已该参数作为名称的Symbol值，有就返回，没有就以这个名称创建一个Symbol值。

```javascript
var s1 = Symbol.for('foo')
var s2 = Symbol.for('foo')
s1 === s2 // true
```

`Symbol.keyFor`方法返回一个已经登记的Symbol类型值的`key`

```javascript
var s1 = Symbol.for("foo");
Symbol.keyFor(s1) // foo

var s2 = Symbol("Foo");
Symbol.keyFor(s2) // undefined
```

### 模块的单例模式

```javascript
const FOO_KEY = Symbol.for('foo');

function A() {
  this.foo = 'hello';
}

if (!global[FOO_KEY]) {
  global[FOO_KEY] = new A();
}

module.exports = global[FOO_KEY];
```

上面的代码中，可以保证`global[FOO_KEY]`不会被无意间覆盖，但还是可以被改写。

## 内置的Symbol值

除了定义自己使用的Symbol值以外，ES6还提供了11个内置的Symbol值，指向语言内部使用的方法。

### Symbol.hasInstance

指向一个内部方法。当其他对象使用`instanceof`运算符，判断是否为该对象的实例时，会调用此方法。例如：

```javascript
class MyClass {
  [Symbol.hasIntance](foo) {
    return foo instanceof Array;
  }
}

[1, 2, 3] instanceof new MyClass() // true
```

### Symbol.isConcatSpreadable

等于一个布尔值，表示该对象使用`Array.prototype.concat()`时，是否可以展开。默认情况下是可以展开的。

```javascript
let arr1 = ['c', 'd'];
['a', 'b'].concat(arr1, 'e'] // ['a', 'b', 'c', 'd', 'e']
arr1[Symbol.isConcatSpreadable] // undefined

let arr2 = ['c', 'd'];
arr2[Symbol.isConcatSpreadable] = false;
['a', 'b'].concat(arr2, 'e'] // ['a', 'b', ['c', 'd'], 'e']
```

### Symbol.species

指向一个方法，该对象作为构造函数创造实例时，会调用这个方法，如果`this.constructor[Symbol.species]`存在，会使用这个属性作为构造函数，来创造新的实例对象。

```javascript
// 默认读取器如下
static get [Symbol.species] {
  return this;
}
```

### Symbol.match

指向一个函数，当执行`str.match(myObject)`时，如果该属性存在，会调用它，返回该方法的返回值。

```javascript
String.prototype.match(regexp);
// 等同于
regexp[Symbol.match](this);

class MyMatcher {
  [Symbol.match](string) {
    return 'hello world'.indexOf(string);
  }
}

'e'.match(new MyMatcher()]; // 1
```

### Symbol.replace

指向一个方法，当该对象被`Symbol.prototype.replace`方法调用时，会返回该方法的返回值。

```javascript
String.prototype.replace(searchValue, replaceValue);
// 等同于
searchValue[Symbol.replace](this, replaceValue);
```

下面有一个例子说明这个问题：

```javascript
const x = {};
x[Symbol.replace] = {...s} => console.log(s);

'Hello'.replace(x, 'World') // ["Hello", "World"]
```

第一个参数是replace方法正在坐拥的对象，第二个参数替换后的值。

### Symbol.search

指向一个方法，当该对象被`String.prototype.search`方法调用时，会返回该方法的返回值。

```javascript
String.prototype.search(regexp);
// 等同于
regexp[Symbol.search](this);

class MySearch {
  constructor(value) {
    this.value = value;
  }
  [Symbol.search](string) {
    return string.indexOf(this.value);
  }
}
'foobar'.search(new MySearch('foo')) // 0
```

### Symbol.split

指向一个方法，当该对象被`String.prototype.split`方法调用时，会返回该方法的返回值。

```javascript
String.prototype.split(separator, limit);
// 等同于
separator[Symbol.split](this, limit);
```

### Symbol.iterator

指向该对象的默认遍历方法

```javascript
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable] // [1, 2, 3]
```

### Symbol.toPrimitive

指向一个方法。该对象被转化为原始类型的值时，会调用这个方法，返回该对象对应的原始类型值。

该方法被调用时，会接受一个字符串参数，表示当前运算的模式，共有三种模式：

- Number：该场合需要转换成数值
- String：该场合需要转换成字符串
- Defaul：该场合既可以转换成数值，也可以转换成字符串

```javascript
let obj = {
  [Symbol.toPrimitive](hint) {
    switch(hint) {
      case 'number':
        return 123;
      case 'string':
        return 'str';
      case 'default':
        return 'default';
      default:
        throw new Error();
    }
  }
};

2 * obj // 246
3 + obj // 3default
obj == 'default' // true
String(obj) // str
```

### Symbol.toStringTag

指向一个方法，在该对象上面调用`Object.prototye.toString`方法时，如果这个属性存在，它的返回值会出现在`toString`方法返回的字符串之中，表示对象的类型。这个属性可以用于定制`[object Object]`或者`[object Array]`中`object`后面的那个字符串。

```javascript
({[Symbol.toStringTag]: 'Foo'}.toString())
// [object Object]

class Collection {
  get [Symbol.toStringTag]() {
    return 'xxx';
  }
}

var x = new Collection();
Object.prototype.toString.call(x); // [object xxx]
```

### Symbol.unscopables

指向一个对象。该对象指定了使用`with`关键字时，哪些属性会被`with`环境排除。

```javascript
Array.prototype[Symbol.unscopables]

Object.keys(Array.prototype[Symbol.unscopables])
```

上面的代码说明，数组有6个属性，会被`with`命令排除。

```javascript
class MyClass {
  foo() {return 1;}
}

var foo = function () {return 2;}

with [MyClass.prototype] {
  foo();
}

class MyClass {
  foo() {return 1;}
  get [Symbol.unscopables]() {
    return {foo: true};
  }
}

var foo = function() {return 2;}
with (MyClass.prototype) {
  foo();
}
```
