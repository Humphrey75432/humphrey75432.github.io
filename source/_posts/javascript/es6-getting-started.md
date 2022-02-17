---
title: 初识 ES6
date: 2022-02-17 10:48:32
updated: 2022-02-17 10:48:32
tags: ES6基础
categories: JavaScript
---

# ES6历史

 ES6全程ECMAScript 6.0，也叫ES2015。ECMA是国际标准化组织，推出这个标准是为了希望此种编程语言能够成为国际标准。因此我们常说的JavaScript实际上是ES的一种实现形式，而ES是JavaScript的规范草案。很多人在一开始学习JavaScript的时候一看到Java就会把它和Sun（Oracle）公司的框架联结起来，实际上两者并没有什么关系。

# 查看Node支持的ES特性

打开终端，前提是需要装好node（为了方便切换node版本你可以安装NVM），然后在控制台输入下列语句：(不同版本的node显示的结果不相同)

```shell
$ node --v8-options | grep harmony

--es-staging (enable test-worthy harmony features (for internal use only))
  --harmony (enable all completed harmony features)
  --harmony-shipping (enable all shipped harmony features)
  --harmony-private-methods (enable "harmony private methods in class literals" (in progress))
  --harmony-regexp-sequence (enable "RegExp Unicode sequence properties" (in progress))
  --harmony-weak-refs (enable "harmony weak references" (in progress))
  --harmony-intl-dateformat-quarter (enable "Add quarter option to DateTimeFormat" (in progress))
  --harmony-intl-add-calendar-numbering-system (enable "Add calendar and numberingSystem to DateTimeFormat")
  --harmony-intl-dateformat-day-period (enable "Add dayPeriod option to DateTimeFormat")
```

# 使用Babel完成ES6解码

 因为ES6草案还在不断完善，而且由于不同平台环境的差异，导致支持ES6的情况都不相同。这个时候Babel可以将我们旧的ES5语法转换成ES6语法，这样在写程序的时候就不用关心底层的环境了。

```javascript
// 转码前语法
input.map(item => item + 1);

// 转码后
input.map(function (item){
  return item + 1;
});
```

## 配置Babel

核心文件就是.babelrc文件，一般在引入Babel后都可以看到这个文件，细则可以去看Babel的官方文档，这里就不扯了。
