---
title: ES6 类
date: 2022-02-17 22:21:01
updated: 2022-02-17 22:21:01
tags: ES6基础
categories: JavaScript
---

# ES6 Class

## 基本概念

ES6为了使JavaScript更接近C++、Java等面向对象的高级语言，引入了Class的概念。语法角度来说Class仅仅是一个语法糖，其绝大部分功能都是ES5的实现。所以是对编程人员友好而设计的。有个例子能充分说明这个问题。

```javascript
// ES5
function Point(x, y) {
    this.x = x;
    this.y = y;
}

Point.prototype.toString = function() {
    return '(' + this.x + ', ' + this.y + ')';
}

var p = new Point(1, 2);

// ES6 更加符合面向对象语言的习惯
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    
    toString() {
        return '(' + this.x + ', ' + this.y + ')';
    }
}
var p = new Point(1, 2);
```

可以看出，调用类的实例方法，实际上就是调用对象的原型方法。那么对Object的操作对于Class完全适用。例如我可以使用Object.assign方法给类新增很多方法。

```javascript
class Point {
    constructor() {
        // ...
    }
}

Object.assign(Point.prototype, {
    toString() {},
    toValue() {}
});
```

## constructor方法

对象的构造函数，会在对象创建的时候调用，可以用来初始化对象的属性。除此之外，和C++/Java等语言不同的是，constructor中完全可以指定返回另外一个对象。

```javascript
class Foo {
    constructor() {
        return Object.create(null);
    }
}

new Foo() instanceof Foo // false
```

## 类的实例对象

需要使用`new`关键字来生成。如果忘记使用new关键字而是直接像使用函数那样使用Class会报错。和ES5一样，除非属性显示定义在其本身，否则都是定义在原型上的。

```javascript
//定义类
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```

## 不存在变量提升

Class不存在变量提升，这一点和ES5完全不同：由于ES6不会把类的声明提升到代码头部。必须保证子类在父类之后定义。

## Class表达式

与函数一样，类也可以使用表达式的形式定义。需要注明的是`const`关键字后面的才是真正的类名，而`class`后面的仅仅是内部类名，可以使用`this`关键字指代。

```javascript
const MyClass = class Me {
    getClassName() {
        return Me.name;
    }
};
```

使用Class表达式，可以立即写出执行的Class

```javascript
let person = new class {
    constructor(name) {
        this.name = name;
    }
    
    sayName() {
        console.log(this.name);
    }
}('ZhangSan');

person.sayName(); // 'ZhangSan'
```

## 私有方法

利用Symbol值的唯一性，将私有方法的名字命名为一个Symbol值，从而达到私有方法和私有属性的效果。

```javascript
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass {
    foo(baz) {
        this[bar](baz);
    }
    
    [bar](baz) {
        return this[snaf] = baz;
    }
    
    // ...
}
```

## this的指向

类的方法内部如果含有this，默认指向类的实例。但是必须非常小心，一旦单独使用该方法，很可能报错。

```javascript
class Logger {
    printName(name = 'there') {
        this.print(`Hello ${name}`);
    }
    
    print(text) {
        console.log(text);
    }
}

const logger = new Logger();
const { printName } = logger;
printName(); // TypeError: Cannot read property 'print' of undefined
```

将printName提取出来使用会报错，因为默认情况下this是指向Logger类的实例。但是一旦将该方法提起出来单独使用，this就会指向该方法运行时所在的环境。为了解决这个问题可以使用下面两种方法。

```javascript
// 在构造函数中重新自动this关键字的指向
class Logger {
    constructor(){
        this.printName = this.printName.bind(this);
    }
}

// 使用箭头函数来改变this关键字的指向
class Logger {
    constructor() {
        this.printName = (name = 'there') => {
            this.print(`Hello ${name}`);
        };
    }
}
```

还有一种方法是使用Proxy，获取方法的时候自动绑定this.

```javascript
function selfish(target) {
    const cache = new WeakMap();
    const handler = {
        get(target, key){
            const value = Reflect.get(target, key);
            if (typeof value !== 'function') {
                return value;
            }
            if (!cache.has(value)) {
                cache.set(value, value.bind(target));
            }
            return cache.get(value);
        }
    };
    const proxy = new Proxy(target, handler);
    return proxy;
}

const logger = selfish(new Logger());
```

## 严格模式

类和模块的内部默认使用的就是严格模式，所以不需要显式指定。

## Class继承

### 基本用法

和其他高级编程语言（C++，Java）一样，ES6也提供了类似`extends`关键字来实现继承。

```javascript
class ColorPoint extends Point {}
```

需要注意的是：在子类的构造函数中，只有调用`super`关键字以后才可以使用`this`关键字，否则会报错。这是因为子类实例的构建是基于对父类的加工，因此只有super方法才能但会父类实例。

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
}

class ColorPoint extends Point {
    constructor(x, y, color) {
        super(x, y);
        ths.color = color;
    }
}
```

## 类的prototype属性和\_\_proto\_\_属性

Class作为构造函数的语法糖，同时有prototype属性和`__proto__`属性，因此同时存在两条继承链。

（1）子类的`__proto__`属性，表示构造函数的继承，总是指向父类；

（2）子类的prototype属性和`__proto__`属性，表示方法的继承，总是指向父类的`prototype`属性。

```javascript
class A {
    
}

class B extends A {
    
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

因此可以这样理解：作为一个对象，子类B的原型(\_\_proto\_\_)是父类A；作为一个构造函数，子类B的原型(prototype.\_\_proto\_\_)是父类A的实例；

## super关键字

`super`既可以当函数使用，也可以当对象使用，两种情况下的使用方法完全不同；

### 当函数使用

这个很简单：作为函数调用时，代表的是父类的构造函数，ES6要求，子类的构造函数必须执行一次super函数。

### 当对象使用

super作为对象时，指向的是父类的原型对象。例子如下：

```javascript
class A {
    p() {
        return 2;
    }
}

class B extends A {
    constructor() {
        super();
        console.log(super.p());
    }
}

let b = new B();
```

这个时候子类B当中的`super.p()`就是将super当做一个对象使用。所以`super`指向的是`A.prototype`，所以`super.p()`就相当于`A.prototype.p()`。需要注意的是：由于`super`指向的是父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过`super`调用的。

```javascript
class A {
    constructor() {
        this.p = 2;
    }
}

class B extends A {
    get m() {
        return super.p;
    }
}

let b = new B();
b.m // undefined
```

如果是定义在父类原型上，`super`就可以取到。

```javascript
class A {}
A.prototype.x = 2;

class B extends A {
    constructor() {
        super();
        console.log(super.x) // 2
    }
}

let b = new B();
```

上面代码中，由于属性x是定义在A.prototype上面的，因此super.x就可以取到它的值。

## Class的取值函数（getter）和存值函数（setter）

和其他高级语言一样，可以使用get、set关键字，对某个属性设置存值和取值函数，拦截该属性的存取行为。

```javascript
class MyClass {
    constructor() {
        // ...
    }
    
    get prop() {
        return 'getter';
    }
    
    set prop(value) {
        console.log('setter: ' + value);
    }
}

let inst = new MyClass();

inst.prop = 123;

inst.prop // 123
```

## Class的Generator方法

如果某个方法之前加上星号，就表示该方法是一个Generator函数。

```javascript
class Foo {
    constructor(...args) {
        this.args = args;
    }
    
    * [Symbol.iterator]() {
        for (let arg of this.arg) {
            yield arg;
        }
    }
}

for (let x of new Foo('hello', 'world')) {
    console.log(x);
}
// hello
// world
```

## Class的静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上`static`关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称之为静态方法。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function
```

父类的静态方法可以被子类继承。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod(); // 'hello'
```

## Class的静态属性和实例属性

静态属性指的是Class本身的属性，即`Class.propname`，而不是定义在实例对象（`this`)上的属性。目前Babel转码器提供了这两种写法的支持。

- 类的实例属性

  类的实例属性可以用等式，写入类的定义之中

  ```javascript
  class ReactCounter extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        count: 0
      };
    }
    state;
  }
  ```

- 类的静态属性

类的静态属性只要在上面的实例属性写法前面，加上static关键字就可以了。

```javascript
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myProp); // 42
  }
}
```

## new.target属性

`new`是从构造函数生成实例的命令，ES6为了`new`命令引入了一个`new.target`属性，返回`new`命令作用于那个构造函数。如果构造函数不是通过`new`命令调用的，`new.target`会返回`undefined`，因此这个属性可以用来确定构造函数是如何调用的。

```javascript
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必须使用new生成实例');
  }
}

// 另一种写法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用new生成实例');
  }
}

var person = new Person('张三'); // 正确
var notAPerson = Person.call(person, '张三');  // 报错
```

## Mixin模式的实现

Mixin模式指的是，将多个类的接口“混入”另一个类，在ES6的实现如下。

```javascript
function mix(...mixins) {
  class Mix {}

  for (let mixin of mixins) {
    copyProperties(Mix, mixin);
    copyProperties(Mix.prototype, mixin.prototype);
  }

  return Mix;
}

function copyProperties(target, source) {
  for (let key of Reflect.ownKeys(source)) {
    if ( key !== "constructor"
      && key !== "prototype"
      && key !== "name"
    ) {
      let desc = Object.getOwnPropertyDescriptor(source, key);
      Object.defineProperty(target, key, desc);
    }
  }
}
```
