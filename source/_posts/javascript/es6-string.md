---
title: ES6 字符串
date: 2022-02-17 11:26:20
updated: 2022-02-17 11:26:20
tags: ES6基础
categories: JavaScript
---

# 字符串

和其他类型的编程语言一样，ES6对原有JavaScript字符的基础上进行了扩展和增强；

## Unicode字符表示法

使用`\uxxxx`的方式表示字符，但是仅仅能表示`\u0000`到`\uFFFF`之间的字符串，超出后必须使用两个字符来表示。

## codePointAt函数

对于使用4个字节存储的字符，JavaScript不能很好地处理。因此ES6提供了codePointAt()方法来处理4个字节的字符。`codePointAt()`会返回32位的UTF-16字符，常规码点和`charAt()`返回值相同。如下所示：

```javascript
var s = "𠮷";
s.length // 2
s.charAt(0); // ''
s.charAt(1); // ''
s.codePointAt(0); // 55362
s.codePointAt(1); // 57271
```

测试一个字符是2字节还是4字节的方法如下：

```javascript
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}
```

## String.fromCodePoint()

ES5开始提供了该方法，用于从码点返回对应字符，但是无法识别UTF-16的字符（也就是Unicode编号大于0xFFFF），但是在ES6开始，该函数可以识别`0xFFFF`的函数。示例如下：

```javascript
String.fromCodePoint(0x20BB7);

String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD830D\uDE80y'
```

## 字符串的遍历接口

ES6为字符提供了遍历接口（因为实现了Iterator接口），使得字符串可以被`for ... of`遍历循环；

```javascript
for (let codePoint of 'foo') {
  console.log(codePoint);
}

// "f"
// "o"
// "o"
```

除了遍历字符串还可以识别大于`0xFFFF`的码点，例如：

```javascript
var text = String.fromCodePoint(0x20BB7);

for (let i of text) {
  console.log(i);
}
```

## at()

ES5中的charAt()方法不能用于识别码点大于`0xFFFF`的字符，但是ES6提案中的`at()`却可以识别码点大于`0xFFFF`的字符。如下所示：

```javascript
	'abc'.at(0)
'0x20BB7'.at(0) 
```

## normalize()

ES6提供的normalize()函数可以将字符的不同表示犯法统一为同样的形式，这样称之为Unicode的正规化；使用正规化的原因时欧洲许多国家的字符存在重音符号。有些字符时通过两个符号合成得到的。但是目前还不支持三个字符的合成分解，所以对于多字符情况还是使用正则表达式吧。

```javascript
'\u01D1'.normalize() === '\u004F\u030C'.normalize() // true
```

此外normalize()方法可以接受一个参数来指定normalize的方式，参数可选值如下：

- `NFC`：默认参数，标准等价合成，返回合成字符；即视觉上和语义上的等价；
- `NFD`：标准等价分解，返回合成字符分解的多个简单字符；
- `NFKC`：兼容等价合成，返回合成字符。即语义上等价、视觉上不等价；
- `NFKD`：兼容等价分解，返回合成字符分解的多个简单字符；

## includes(), startsWith(), endsWith()

传统的JavaScript只有一个`indexOf()`方法用来确定一个字符串是否包含在另一个字符串中。ES6又提供了三种新方法：

- `includes()`：表示是否找到了参数字符串
- `startsWith()`：表示参数字符串是否在源字符串的头部；
- `endsWith()`：表示参数字符串是否在源字符串的尾部；

例子如下：

```javascript
var s = "Hello World!";

s.startsWith('Hello'); // true
s.endsWith('!'); // true
s.includes('o'); // true
```

## repeat()

`repeat()`方法返回一个新字符串，表示将源字符串重复n次，如果n为小数则会向下取整；如果为负数或者`Infinity`会报`Range Error`错误；如果时-1~0之间的数据会先取整再运算，因此当0处理；

```javascript
'x'.repeat(3); //xxx
'hello'.repeat(2); //hellohello
'na'.repeat(0); // ""
'na'.repeat(2.9); // "nana"
'na'.repeat(Infinity); // Range Error
'na'.repeat(-1); // Range Error
'na'.repeat(-0.5); // ""
```

## padStart(), padEnd()

ES7推出字符串补全长度功能。如果某个字符串不够指定长度，这会在头部和尾部补全。此处不再赘述。

```javascript
'x'.padStart(5, 'ab'); // 'ababx';
'x'.padStart(4, 'ab'); // 'abax';

'x'.padEnd(5, 'ab'); // 'xabab';
'x'.padEnd(4, 'ab'); // 'xaba';
```

如果原字符串的长度，等于或大于指定的最小长度，则返回原字符串。

```javascript
'xxx'.padStart(2, 'ab'); // xxx
```

如果用来补全的字符串与原字符串，两者的长度之和超过了指定的最小长度，则会截去超出位数的补全字符串；如果省略第二个字符，则会使用空格来进行填充。

```javascript
'abc'.padStart(10, '0123456789'); // 0123456abc
'X'.padStart(4); '   X'
```

因此也可以看出它主要是用来给数值补全指定位数或者字符串格式化。例子如下：

```javascript
// 数值补位
'1'.padStart(10, '0'); // 0000000001
'12'.padStart(10, '0'); // 0000000012
'123456'.padStart(10, '0'); // 0000123456

// 字符串格式
'12'.padStart(10, 'YYYY-MM-DD'); // YYYY-MM-12
'09-12'.padStart(10, 'YYYY-MM-DD'); // YYYY-09-12
```

## 模板字符串

ES6引用了模板字符串来撰写模板，这样可以在表示上更加简洁。类似的语法在Python和Golang中也有；如下所示：使用`${expression}`包裹的表达式可以作为占位符解析：

```javascript
$('#result').append(`
	There are <b>${basket.count}</b> items
	in your basket, <em>${basket.onSale}</em>
	are on sale!
`);
```

所有模板字符的空格和换行都是被原样保留的，如果不想要换行，可以加上`trim()`来消除：

```javascript
$('#list').html(`
<ul>
	<li>first</li>
	<li>second</li>
</ul>
`.trim());
```

模板字符串除了可以嵌套JavaScript表达式，还可以调用函数；除此之外，模板表达式还可以嵌套。注意：由于模板字符串可以传入JavaScript，因此变量在使用前必须声明，否则会编译失败；

```javascript
const tmpl = addrs => `
	<table>
	${addrs.map(addr => `
		<tr><td>${addr.first}</td></tr>
		<tr><td>${addr.last}</td></tr>
	`).join('')}
	</table>
`;
```
