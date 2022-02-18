---
title: CSS3之Sass学习
date: 2022-02-18 20:50:30
updated: 2022-02-18 20:50:30
tags: CSS3
categories: 前端
---

# Sass 基本教程

## 简介

Sass一个CSS预处理器，属于CSS的扩展语言，因此语法完全兼容CSS；最早是由Hampton Catlin设计并由Natalie Weizenbaum开发的层叠样式表语言。Sass全称（Syntactically Awesome Stylesheets）

Sass的文件后缀为`.scss`，因此Scss生成的格式化良好的代码，易于组织和维护；出现Scss的原因是CSS原本的语法不够强大，导致重复编写一些代码，无法实现复用，而且维护起来也不方便；

## 环境安装

安装也非常简单，可以用NPM来安装，如果是苹果用户可以用Homebrew；

```sh
$ npm install -g sass

$ brew install sass/sass/sass
```

## 语法

### 基础示例

安装好scss后就可以使用了，可以贴上一个例子：

```scss
$bgcolor: lightblue;
$textcolor: darkblue;
$fontsize: 18px;

body {
    background-color: $bgcolor;
    color: $textcolor;
    font-size: $fontsize;
}
```

### 变量

变量用于存储一些信息，它可以重复使用，可以存储以下信息：

- 字符串
- 数字
- 颜色值
- 布尔值
- 列表
- null值

Sass变量使用`$`符号，Sass变量的作用域只能在当前层级上有效果

如果要设置全局作用域，可以使用`!global`关键词来设置，例子如下：

```scss
$myColor: red;

h1 {
    $myColor: green !global;
    color: $myColor;
}
```

## 嵌套规则与属性

Sass嵌套CSS选择器类似于HTML的嵌套规则，例如：

```scss
nav {
    ul {
        margin: 0;
        padding: 0;
        list-style: none;
    }
    
    li {
        display: inline-block;
    }
    
    a {
        display: block;
        padding: 6px 12px;
        text-decoration: none;
    }
}
```

很多CSS属性都有同样的前缀，例如font-family，font-size和font-weight，text-align，text-transform和text-overflow，在Sass中可以使用嵌套属性来编写，例如：

```scss
font: {
    family: Helvetica, sans-serif;
    size: 18px;
    weight: bold;
}

text: {
    align: center;
    transform: lowercase;
    overflow: hidden;
}
```

以上代码会被转换成如下：

```css
font-family: Helvetica, sans-serif;
font-size: 18px;
font-weight: bold;

text-align: center;
text-transform: lowercase;
text-overflow: hidden;
```

## @import 和 Partials

### @import

Sass可以帮助减少重复的CSS代码，节省开发时间。

类似CSS，Sass支持`@import`指令，可以让我们导入其他文件等内容；例如：

创建一个`reset.scss`文件

```scss
html,
body,
ul,
ol {
    margin: 0;
    padding: 0;
}
```

然后在`standard.scss`文件中使用`@import`指令导入`reser.scss`文件；

```scss
@import "reset";

body {
    font-family: Helvetica, sans-serif;
    font-size: 18px;
    color: red;
}
```

### Partials

如果你不希望将一个Sass代码编译到CSS文件中，可以在文件名的开头添加一个下划线。这将告诉Sass不要将Sass编译到CSS文件中。

例如我创建一个`_colors.sass`文件，但是不会被编译成`_colors.css`；

```scss
$myPink: #ee82ee;
$myBlue: #4169e1;
$myGreen: #8fbc8f;
```

如果需要导入该文件，则不需要使用下划线；

```scss
@import "colors";

body {
    font-family: Helvetica, sans-serif;
    font-size: 18px;
    color: $myBlue;
}
```

需要注意的是：不要将待下划线与不带下划线的同名文件放置在同一个文件目录下，比如：`_colors.scss`和`colors.scss`不能同时存在于一个目录下，否则带下划线的文件将会被忽略；

## @mixin 和 @include

`@mixin`指令允许我们定义一个可以在争个光样式表中重复使用的样式，语法为`@mixin name {property: value}`

```scss
@mixin important-text {
    color: red;
    font-size: 25px;
    font-weight: bold;
    border: 1px solid blue;
}
```

`@include`指令可以将混入（mixin）引入文档中；同样地，包含important-text混入代码如下：

```scss
.danger {
    @include important-text;
    background-color: green;
}
```

除此之外，混入可以接收参数，我们可以向混入传递变量，定义可以接收参数的混入：

```scss
@mixin bordered($color, $width) {
    border: $width solid $color;
}

.myArticle {
    @include bordered(blue, 1px);
}

.myNotes {
    @include bordered(red, 2px);
}
```

除此之外，混入也可以定义默认值，语法格式如下：

```scss
@mixin bordered($color: blue, $width: 1px) {
    border: $width solid $color;
}
```

有时候不能确定混入（mixin）或者一个函数（function）使用多少个参数，这个时候我们就可以使用`...`来设置可变参数，例如：

```scss
@mixin box-shadow($shadows...) {
    -moz-box-shadow: $shadows;
    -webkit-box-shadow: $shadows;
    box-shadow: $shadows;
}

.shadows {
    @include: box-shadow(0px 4px 5px #666, 2px 6px 10px #999);
}
```

浏览器前缀使用混入也是十分方便的；

```scss
@mixin transform($property) {
    -webkit-transform: $property;
    -ms-transform: $property;
    transform: $property;
}

.myBox {
    @include: transform(rotate(20deg));
}
```

## @extend 和继承

`@extend`指令告诉Sass一个选择器的样式从另一个选择器继承；

如果一个样式与另一个样式几乎相同，只有少量区别，则使用@extend就显得更有用；例如：

```scss
.button-basic {
    border: none;
    padding: 15px 30px;
    text-align: center;
    font-size: 16px;
    cursor: pointer;
}

.button-reporter {
    @extend .button-basic;
    background-color: red;
}

.button-submit {
    @extend .button-basic;
    background-color: green;
    color: white;
}
```

通过将公共代码抽离出来公用给其他CSS样式从而实现代码的复用，真棒；

## Sass函数

Sass定义了各种类型的函数，可以通过CSS语句直接调用；这里就不再赘述，有需要可以自行查询文档了解；

- 字符串相关函数
- 数字相关函数
- 列表相关函数
- 映射相关函数
- 选择器相关函数
- Introspection相关函数
- 颜色相关函数
