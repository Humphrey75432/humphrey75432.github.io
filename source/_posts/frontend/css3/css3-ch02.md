---
title: CSS3进阶操作
date: 2022-02-18 20:50:27
updated: 2022-02-18 20:50:27
tags: CSS3
categories: 前端
---

# CSS3 进阶操作

## CSS3 边框

在CSS3中你可以创建圆角边框、阴影边框。属性如下：

- border-radius：边框圆角
- box-shadow：盒子阴影
- border-image：边框图片

## CSS3 圆角

举个例子：

```css
#cornenr1 {
  border-radius: 25px;
  background: #a8bcf4;
  padding: 20px;
  width: 200px;
  height: 150px;
}

#corner2 {
  border-radius:15px 50px 30px 5px;
  background: #fe5cb9;
  padding: 10px;
  width: 150px;
  height: 100px;
}
```

可以发现第二个例子有四个值：顺序为：左上、右上、右下和左下；

一个值就是默认四个角都是一样的，没什么毛病。

初次之外还有其他属性，可以自己熟悉一下：

- border-top-left-radius
- border-top-right-radius
- border-bottom-left-radius
- border-bottom-right-radis

## CSS3 背景

CSS3中新增几个背景属性，提供了更大背景元素控制；

- background-image：添加背景图片，不同背景用逗号分隔开；
- background-size：指定背景图像的大小；
- background-origin：指定背景图像的位置区域，有：border-box, padding-box和content-box;
- background-clip：从指定位置开始绘制背景图形；

## CSS3 渐变

可以在两个或者多个指定的颜色之间显示平稳的过渡，CSS3中定义了两种类型的渐变（gradients）：

- 线性渐变：向上下左右对角方向
- 径向渐变：由中心定义

语法如下：

```css
background-image: linear-gradient(direction, color-stop1, color-stop2, ...)

/* 
  direction: 默认上下，左右为to right
  对角渐变：to bottom right
  还可以使用角度：angle (这里指的是弧度)
*/
```

重复的线性渐变

```css
background-image: repeating-linear-gradient(red, yellow 10%, green 20%);
```

径向渐变

```css
background-image: radial-gradient(shape size at position, start-color, ... ,last-color);
```

shape参数定义了形状。可以是circle和ellipse，圆形和椭圆形；

和重复地线性渐变一样，径向渐变也有重复地属性，这里就不再演示了。

## CSS3 文本效果 & 字体

文本效果包括下面的这几个：

- text-shadow：文本阴影，可以定义多个，可以做字体发光效果
- box-shadow：盒子阴影，可以定义多个，做成发光盒子
- text-overflow：指定用户如何显示溢出内容；
- word-wrap：自动换行属性允许强制文本换行，这意味着分裂它中间的一个字；
- word-break：单词拆分换行属性指定换行规则；

需要使用自定义字体可以使用`@font-face`属性。例子如下：

```css
@font-face {
  font-family: myFirstFont;
  src url(font.ttf);
}

div {
  font-family: myFirstFont;
}
```

## CSS3 2D & 3D转换

### 2D转换

2D变换方法有下面5种：

- translate：根据x轴和y轴位置给定的参数，从当前元素位置移动；
- rotate：在一个给定读书顺时针旋转元素，负值代表逆时针旋转；
- scale：该元素增加或者减少大小，取决于宽度（x轴）和高度（y轴）的参数；
- skew：表示x轴和y轴倾斜的角度，如果第二个参数为空，则默认为0，参数为负数表示像反方向倾斜；
- matrix：和2D变换方法合并成一个，一共有六个参数，包括：旋转、缩放、移动和倾斜的功能；

具体语法可以再查阅

### 3D转换

3D转换会使用到两个方法：

- rotateX：围绕其在一个给定度数X轴旋转的元素；
- rotateY：围绕其在一个给定读书Y轴旋转的元素；

你也可以使用transform属性做2D和3D变换，一般来说建议使用transform属性，特别是在使用一些库来实现高级动画效果的时候；

```css
div {
    transform: none | transform-functions;
}
/*
transform-functions包括：
matrix, matrix3d;
translate, translate3d, translateX, translateY, translateZ;
scale, scale3d, scaleX, scaleY, scaleZ;
rotate, rotate3d, rotateX, rotateY, rotateX;
skew, skewX, skewY;
perspective：为3D转换元素定义透视视图；
*/
```

## CSS3 过渡

CSS3过渡是元素从一种样式逐渐改变为另一种效果，要实现这一点，必须规定两项内容；

- 指定要添加效果的CSS属性；
- 指定效果的持续时间；

例子如下：

```css
div {
    transition: width 2s;
    -webkit-transition: width 2s; /* Safari or Chrome */
}

/* 要添加多个样式的变换效果，可以使用逗号分隔属性 */
div {
    transition: width 2s, height 2s, transform 2s;
    -webkit-transform: width 2s, height 2s, transform 2s;
}
```

## CSS3 动画

CSS3可以创建动画，可以取代一些JavaScript实现的动画效果；

CSS3使用`@keyframes`创建动画，语法如下：

```css
@keyframes myfirst {
    from { background: red; }
    to { background: yellow; }
}

@-webkit-keyframes myfirst {
    from { background: red; }
    to { background: yellow; }
}

div {
    animation: myfirst 5s;
}

/* 除此之外还可以这样定义 */
@keyframes myfirst {
    0% { background: red; }
    25% { background: yellow; }
    50% { background: blue; }
    100% { background: green; }
}

@-webkit-keyframes myfirst {
    0% { background: red; }
    25% { background: yellow; }
    50% { background: blue; }
    100% { background: green; }
}

div {
    animation: myfirst 5s;
}
```

## CSS3媒体查询

顾名思义，针对不同媒体类型的设备定制不同的样式规则，例如：手机、电视、平板、电脑以及阅读器设置不同的样式规则；语法如下：

```css
@media not | only mediatype and (expression) {
    /* Write CSS Code here... */
}
```

解释一下参数的含义：

- not：用于排除某些特定设备的；
- only：用来指定某种特别的媒体设备；
- all：适用于所有媒体设备，这个最常见；

举个例子：

```css
@media screen and (min-width: 480px) {
    #leftsidebar { width: 200px; float: left; }
    #main { margin-left: 216px; }
}
```

以上实例在屏幕可视窗口尺寸大于480px像素时将菜单浮动到页面左侧；

## CSS3 Flex布局

弹性布局是CSS3的一种新布局模式，当页面需要适应不同的屏幕大小以及设备类型时确保元素拥有恰当的行为布局方式，引入弹性布局模型的目的是提供一种更加有效地方式来对一个容器中的元素进行排列、对齐和分配空白空间；

下面以例子来说明：

```html
<html>
    <head>
        <style type="text/css">
            .flex-container {
                display: flex;
                width: 400px;
                height: 250px;
                background: lightgray;
            }
            
            .flex-item {
                background-color: cornflowerblue;
                width: 100px;
                height: 100px;
                margin: 10px;
            }
        </style>
    </head>
    
    <body>
        <div class="flex-container">
            <div class="flex-item">flex item 1</div>
            <div class="flex-item">flex item 2</div>
            <div class="flex-item">flex item 3</div>
        </div>
    </body>
</html>
```

### 弹性容器布局属性

#### flex-direction

指定了弹性子元素在父容器中的位置，值有：

- row：默认值，从左到右排列；
- row-reverse：反转横向排列，从右到左排列；
- column：纵向排列；
- column-reverse：反转纵向排列，从后往前排；

#### justify-content

内容对齐属性应用在弹性容器上，把弹性项沿着弹性容器的主轴线对齐，值有：

- flex-start：弹性项目向行头紧挨着填充。这个是默认值。第一个弹性项的main-start外边距边线被放置在该行的main-start边线，而后续弹性项依次平齐摆放。
- flex-end：弹性项目向行尾紧挨着填充。第一个弹性项的main-end外边距边线被放置在该行的main-end边线，而后续弹性项依次平齐摆放。
- center：弹性项目居中紧挨着填充。（如果剩余的自由空间是负的，则弹性项目将在两个方向上同时溢出）。
- space-between：弹性项目平均分布在该行上。如果剩余空间为负或者只有一个弹性项，则该值等同于flex-start。否则，第1个弹性项的外边距和行的main-start边线对齐，而最后1个弹性项的外边距和行的main-end边线对齐，然后剩余的弹性项分布在该行上，相邻项目的间隔相等。
- space-around：弹性项目平均分布在该行上，两边留有一半的间隔空间。如果剩余空间为负或者只有一个弹性项，则该值等同于center。否则，弹性项目沿该行分布，且彼此间隔相等（比如是20px），同时首尾两边和弹性容器之间留有一半的间隔（1/2*20px=10px）。

#### align-items

设置或检索弹性盒子元素在侧轴（纵轴）方向上的对齐方式；值有：

- flex-start：弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴起始边界。
- flex-end：弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴结束边界。
- center：弹性盒子元素在该行的侧轴（纵轴）上居中放置。（如果该行的尺寸小于弹性盒子元素的尺寸，则会向两个方向溢出相同的长度）。
- baseline：如弹性盒子元素的行内轴与侧轴为同一条，则该值与'flex-start'等效。其它情况下，该值将参与基线对齐。
- strench：如果指定侧轴大小的属性值为'auto'，则其值会使项目的边距盒的尺寸尽可能接近所在行的尺寸，但同时会遵照'min/max-width/height'属性的限制。

#### flex-wrap

指定弹性盒子的子元素换行方式，值有：

- nowrap：默认， 弹性容器为单行。该情况下弹性子项可能会溢出容器。
- wrap：弹性容器为多行。该情况下弹性子项溢出的部分会被放置到新行，子项内部会发生断行
- wrap-reverse：反转 wrap 排列。

#### align-content

用于修改flex-wrap属性的行为，类似align-items，但它不是设置弹性子元素的对齐，而是设置各个行的对齐，值有：

- strench：默认。各行将会伸展以占用剩余的空间。
- flex-start：各行向弹性盒容器的起始位置堆叠。
- flex-end：各行向弹性盒容器的结束位置堆叠。
- center：各行向弹性盒容器的中间位置堆叠。
- space-between：各行在弹性盒容器中平均分布。
- space-around：各行在弹性盒容器中平均分布，两端保留子元素与子元素之间间距大小的一半。

### 弹性元素布局属性

#### 排序order

用整数值来定义排列顺序，数值小的排在前面，可以为负值；例子如下：

```css
.flex-item {
    background-color: cornflowerblue;
    width: 100px;
    height: 100px;
    margin: 10px;
}

.flex-item:first-child {
    order: -1;
    -webkit-order: -1;
}
```

#### 对齐

设置margin值为auto，自动获取弹性容器中剩余空间，因此设置垂直方向margin值为'auto'

```css
.flex-item {
    background-color: cornflowerblue;
    width: 75px;
    height: 75px;
    margin: 10px;
}

.flex-item:first-child {
    margin-right: auto;
}
```

#### 完美居中

可以看出，使用弹性盒子后，居中变得非常简单。下面演示一下完美居中的实现方法；

```css
.flex-item {
    background-color: cornflowerblue;
    width: 75px;
    height: 75px;
    margin: auto;
}
```

#### align-self

用于设置弹性元素自身在纵轴方向上的对齐方式，值有：

- auto：如果'align-self'的值为'auto'，则其计算值为元素的父元素的'align-items'值，如果其没有父元素，则计算值为'stretch'。
- flex-start：弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴起始边界。
- flex-end：弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴结束边界。
- center：弹性盒子元素在该行的侧轴（纵轴）上居中放置。（如果该行的尺寸小于弹性盒子元素的尺寸，则会向两个方向溢出相同的长度）。
- baseline：如弹性盒子元素的行内轴与侧轴为同一条，则该值与'flex-start'等效。其它情况下，该值将参与基线对齐。
- strench：如果指定侧轴大小的属性值为'auto'，则其值会使项目的边距盒的尺寸尽可能接近所在行的尺寸，但同时会遵照'min/max-width/height'属性的限制。

以下演示了不同值的效果，可以在[Code Pen](https://codepen.io)上直接试验：

```css
.flex-item {
    background-color: cornflowerblue;
    width: 60px;
    min-height: 100px;
    margin: 10px;
}

.item1 {
    align-self: flex-start;
    -webkit-align-self: flex-start;
}

.item2 {
    align-self: flex-end;
    -webkit-align-self: flex-end;
}

.item3 {
    align-self: center;
    -webkit-align-self: center;
}

.item4 {
    align-self: baseline;
    -webkit-align-self: baseline;
}

.item5 {
    align-self: strench;
    -webkit-align-self: strench;
}
```

#### flex

用于指定弹性子元素如何分配空间，值有：

- auto：计算值为1 1 auto
- initial：计算值为0 1 auto
- none：计算值为0 0 auto
- inherit：从父元素继承
- flex-grow：定义弹性盒子元素的扩展比率
- flex-shrink：定义弹性盒子元素的搜索比率
- flex-basis：定义弹性盒子元素的默认基准值

例子：

```css
.flex-item {
    background-color: cornflowerblue;
    margin: 10px;
}

.item1 {
    -webkit-flex: 2;
    flex: 2;
}

.item2 {
    -webkit-flex: 1;
    flex: 1;
}

.item3 {
    -webkit-flex: 1;
    flex: 1;
}
```
