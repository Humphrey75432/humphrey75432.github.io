---
title: CSS3基础回顾
date: 2022-02-18 20:50:23
updated: 2022-02-18 20:50:23
tags: CSS3
categories: 前端
---

# CSS3 基础回顾

## 简介

全称“层叠样式表”（Cascade Style Sheet），用来修饰HTML元素的外观：包括大小、位置、颜色和形状等等；

## 语法

语法如下：

```css
selector {
  property-type: property-value;
}
```

简单来说：需要对哪个元素做修饰，首先要选中需要修饰的元素，然后再使用CSS对其作修改。

## 选择器

### 基本

比较常用的选择器有：class选择器和id选择器，除此之外还有一些高级的选择器，在日常使用中我们可以通过查阅文档来了解一些更高级的选择器用法。

- class选择器
- id选择器
- nth-child选择器等

一个例子来说明这些选择器的用法：

```css
.className {
  width: 200px;
  height: 150px;
}

#id-name {
  width: 200px;
  height: 150px;
}

.box:nth-child(2n) {
  background: #4f5b62;
}
```

上面列举了三个选择器的语法：可以看到class选择器的语法是`.className`，id选择器的语法是`#idName`；掌握这两个基本就能满足90%以上的应用场景了；

最后还有一个选择器值得关注：它的语义是：选择className为box并且顺序为偶数序的元素。这属于高级选择器中的其中一个，还有其他的一些可以通过上网查阅来了解；

一般在前端开发中会遵循一个编码规范：就是class选择器专门用来修饰样式；id选择器专门用来编写逻辑；

### 属性选择器

具有特定属性的HTML元素样式不仅仅是class和id，语法如下：

```css
[title] {
    color: blue;
}
```

举个例子：

```css
input[type="text"] {
    width: 150px;
    display: block;
    margin-bottom: 10px;
    background-color: yellow;
}

input[type="button"] {
    width: 120px;
    margin-left: 35px;
    display: block;
}
```

### 嵌套和分组

给很多样式相同的元素设置样式，可以使用分组选择器，语法如下：

```css
h1,
h2,
p {
  color: green;
}
```

嵌套选择器可以用于选择器内部的样式，有下面几种实践：

- `p{}`：为所有p元素指定一个样式；
- `.marked {}`：为所有`class="marked"`的元素设置样式；
- `.marked p {}`为所有`class="marked"`元素内部的`p`元素设置样式；
- `p.marked {}`为所有`p`元素内部`class="marked"`的元素设置样式；

## 盒子模型

所有的HTML元素都可以看作是盒子，在CSS中，“box model”是用来设计和布局使用的；

一个盒子有下列几部分构成：

- 外边距(margin)：清除边框外的区域，外边距是透明的；
- 边框(border)：围绕在内边距和内容外的边框，这部分是可以看见的；
- 内边距(padding)：清除内容周围的区域，内边距是透明的；
- 内容(content)：盒子的内容，显示文本和图像；

所以你需要知道：给一个元素设置宽高的时候要加上它的外边距、边框、内边距的宽高，这样才是一个元素的宽高；

盒子模型可以通过浏览器的开发者工具看到，如果需要检视页面上的HTML元素，你可以通过查看元素在控制台的信息来查看它的边距情况。

总结一下：最终元素的总宽高计算公式是这样的；

- 总元素的宽度：宽度 + 左右填充 + 左右边框 + 左右边距；
- 总元素的高度：高度 + 上下填充 + 上下边框 + 上下边距；

## 样式设置

### 颜色

可以给字体，盒子设置颜色，下面举一部分例子：

- 盒子背景：`background-color`，背景有一个简写的属性叫`background`;
- 字体颜色：`font-color`，同样地，字体也有一个简写属性叫`font`;
- 边框：边框使用`border`属性也可以给其上色，具体语法可以查阅相关文档；
- 还有一个CSS属性叫`color`，这个是给文本设置颜色的；

### 字体

字体是HTML中的重要显示元素，CSS中字体的设置有下列几个：

- font：all in one属性，可以设置字体的所有属性
- font-family：字体
- font-size：字体大小
- font-style：字体样式，粗体和斜体之类的
- font-variant：以小型大写字体或者正常字体显示文本；
- font-weight：指定字体的粗细；

介绍一下all-on-one属性的语法：

```css
font: "<font-style> <font-variant> <font-weight> <font-size> / <line-height> <font-family>"
```

### All-in-one 属性语法

#### 外边距（margin）

语法：

```css
h1 {
  marign: 15px 30px 15px 30px; // 次序分别为上、右、下、左
}

h1 {
  margin: 15px 30px 15px; // 次序分别为上，左右，下
}

h1 {
  margin: 15px 30px; // 上下，左右
}

h1 {
  margin: 15px; //上下左右全都一样
}
```

#### 填充（padding）

语法：

```css
h1 {
  padding: 15px 30px 15px 30px; // 次序分别为上、右、下、左
}

h1 {
  padidng: 15px 30px 15px; // 次序为上，左右，下
}

h1 {
  padding: 15px; 30px; // 上下，左右
}

h1 {
  padding: 25px; // 上下左右全都一样
}
```



## 显示

display设置一个元素该如何在页面上显示，visibility属性指定一个元素应可见还是隐藏；

#### 隐藏元素

这里提供两种方法：

- `display: none`：除了会隐藏，还会清除元素原本占用的页面空间；
- `visibility: hidden`：虽然也能隐藏，但是并不会清除元素原本占用的页面空间；

#### 块级元素和内联元素

需要注意的是：块级元素默认占用一行，元素在页面上垂直排列；可以设置宽度和高度；

内联元素则相反：其大小由内容决定，只有填满了一行才会在换行继续，默认情况下不能设置宽和高；但是可以设置边距和填充并且只在水平方向有效；

了解了上面的知识以后，我们就会知道，使用display方法可以改变元素的显示方式是以块级来显示还是以内联方式来显示。相应的属性如下：

- block
- inline

举个例子：加入你想给`<span>`设置元素宽高，但由于其为内联元素，无法设置宽高；这个时候你可以使用display属性为其设置显示模式为`block`，然后就可以给`<span>`设置宽高了；

## 定位

定位属性叫`position`，其指定了元素的定位类型，属性有5个值；

- static：默认设置，没有定位，遵循正常的文档流（即从上到下，从左到右）
- relative：相对定位元素的定位是相对其正常位置
- fixed：元素的位置相对于浏览器窗口是固定的
- absolute：相对于最近的已定位父元素，如果元素没有已定位的父元素，那么它的位置相对于`html`；
- stickly：基于用户的滚动位置来定位；

除此之外，还有一个元素叫做`z-index`，其指定了一个元素的堆叠顺序（哪个元素该放在前面，哪个元素该放在后面），一个元素可以有正数和负数的堆叠顺序；

## 溢出

CSS overflow属性用于控制内容溢出元素框时的显示方式，有下面5个属性值；

- visible：默认值，内容不会被裁剪，会显示在元素框之外
- hidden：内容会被裁剪，并且其余部分的内容是不可见的；
- scroll：内容会被裁剪，但是浏览器会显示滚动条以便于查看其余内容
- auto：如果内容被裁剪，浏览器会显示滚动条以便于查看其余内容
- inherit：规定应该从父元素继承overflow属性的值

## 浮动

浮动的框可以向左或者向右移动，直到它的外边框边缘碰到包含框或另一个浮动框的边框为止。由于浮动框不在文档流的普通流中，所以文档的普通流中的浮动框之后的块框表现地像浮动框不存在一样。

需要注意的是：当初float被设计的时候就是用来完成文本环绕的效果，所以文本不会被挡住，这是float的特性，因此float是一种不彻底的脱离文档流方式。无论多么复杂的布局，其基本出发点均是：“如何在一行显示多个div元素”。

于是在浮动布局中会存在下面这个现象：

- 如果元素A是浮动元素并且其上一个元素也是浮动元素，那么A元素就会跟随在上一个元素的后边；（如果一行放不下就会挤到下一行）
- 如果元素A是浮动元素，但是其上一个元素是标准流元素，那么A的相对垂直位置不会改变，也就是说A的顶部总是和上一个元素的底部对齐；
- 浮动框之后的block元素会认为这个框不存在，但其中的文本依然会认为这个元素让出位置。浮动的框之后的inline元素，会为这个框空出位置，然后按顺序排列；

使用浮动的原则如下：

要么都使用浮动，要么都不使用浮动；要么对没使用浮动的DIV设置margin样式；

## 清除浮动

在非IE浏览器的环境下，当容器高度为auto，且容器内容有浮动的元素，在这种情况下，容器的高度不能自动伸长以适应内容的高度，使得内容溢出到容器外面而影响（甚至是破坏）布局的现象。这个现象就叫做浮动溢出，为了防止这个现象出现而进行的CSS处理，就叫做CSS清除浮动。

引用W3C的一个例子来解释浮动溢出：

```html
<html>
    <head>
        <style type="text/css">
            .news: {
                background-color: gray;
                border: 1px solid black;
            }
            
            .news img {
                float: left;
            }
            
            .news p {
                float: right;
            }
        </style>
    </head>
    <!--虽然从HTML上看news是包裹着图片和文字的-->
    <div class="news">
        <!--实际上由于浮动元素不占据空间，所以news的高度直接塌陷了-->
        <img src="news-pic.jpg"/>
	    <p>some text</p>
    </div>
</html>
```

既然出现了元素的“高度塌陷”，那么可以使用清除浮动的方法来解决这个问题：

**方法1：在浮动元素后使用一个空元素，并在CSS中赋予`.clear {clear: both}`属性即可清除浮动；例如：**

```html
<html>
    <head>
    </head>
    
    <body>
        <div class="news">
            <img src="news-pic.jpg"/>
            <p>Some Text</p>
            <div class="clear"/>
        </div>
    </body>
</html>
```

优点：简单、代码少，浏览器兼容性好；

缺点：需要添加大量无语义的html元素，代码不够优雅，后期也不容易维护；



**方法2：使用CSS的overflow属性（BFC机制）**

给浮动元素的容器添加`overflow: hidden`或者`overflow: auto`可以清除浮动，在添加了相关属性后，浮动元素又回到了容器层，把容器高度撑起，达到清除浮动的效果。

```html
<html>
    <head>
        <style type="text/css">
            .news {
                background-color: gray;
                border: 1px solid black;
                overflow: hidden;
                /* 触发IE6 / IE7的haslayout机制 */
                *zoom: 1;
            }
            
            .news img {
                float: left;
            }
            
            .news p {
                float: right;
            }
        </style>
    </head>
    <body>
        <div class="news">
            <img src="news-pic.jpg"/>
            <p>some text</p>
        </div>
    </body>
</html>
```



**方法3：给浮动元素的容器添加浮动**

给浮动元素的容器添加浮动属性即可清除内部浮动，但是这样做会使其整体浮动，影响布局，不推荐使用；



**方法4：使用邻接元素处理**

什么都不做，给浮动元素后面的元素添加clear属性；

```html
<html>
    <head>
        <style type="text/css">
            .news {
                background-color: gray;
                border: 1px solid black;
            }
            
            .news img {
                float: left;
            }
            
            .news p {
                float: right;
            }
            
            .content {
                clear: both;
            }
        </style>
    </head>
    
    <body>
        <div class="news">
            <img src="news-pic.jpg"/>
            <p>Some text</p>
            <div class="content"></div>
        </div>
    </body>
</html>
```



**方法5：使用CSS的:after伪元素**

给浮动元素的容器添加一个clearfix的class，然后给这个class添加一个`:after`伪元素实现方法1的那种效果。此方法兼容性好；推荐这种方法；

```html
<html>
    <head>
        <style type="text/css">
            .news {
                background-color: gray;
                border: 1px solid black;
            }
            
            .news img {
                float: left;
            }
            
            .news p {
                float: right;
            }
            
            .clearfix:after {
                content: "020";
                display: block;
                height: 0;
                clear: both;
                visibility: hidden;
            }
            
            .clearfix {
                /* 触发 IE6/IE7的haslayout机制 */
                zoom: 1;
            }
        </style>
    </head>
    
    <body>
        <div class="news clearfix">
            <img src="news-pic.jpg"/>
            <p>Some text</p>
        </div>
    </body>
</html>
```

## 伪类

CSS伪类是用来添加一些选择器的特殊效果，语法如下：

```css
/* 基础语法 */
selector:pseudo-class { property: value; }

/* CSS类选择器 */
selector.class:pseudo-class { property: value; }
```

### anchor伪类

链接的不同状态都可以以不同的方式显示，例如：

- `a:link`：未访问的链接
- `a:visited`：已访问的链接
- `a:hover`：鼠标划过的链接
- `a:active`：已选中的链接

### CSS伪元素

CSS伪元素用来添加一些选择器的特殊效果，语法如下：

```CSS
/* 伪元素语法 */
selector:pseudo-element { property: value; }

/* CSS类也可以使用伪元素 */
selector.class:pseudo-element { property: value; }
```

下面列举一些伪元素的使用：

- `:first-line`：用于向文本的首行设置特殊样式；
- `:first-letter`：用于向文本的首字母设置特殊样式；
- `:before`：可以在元素的内容前插入新内容；
- `:after`：可以在元素的内容后插入新内容；
