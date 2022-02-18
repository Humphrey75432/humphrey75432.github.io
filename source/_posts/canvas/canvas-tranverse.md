---
title: Canvas变换
date: 2022-02-18 10:59:34
updated: 2022-02-18 10:59:34
tags: HTML
categories: 前端
---

# Canvas变换

## 图形变换

图形变换无外乎是利用数学方法调整所绘形状的物理属性，实质上是坐标变形。所有的变换都依赖于后台的数学矩阵运算，所以只需要使用即可，无需理会底层原理。

- 平移变换：`translate(x, y)`
- 旋转变换：`rotate(deg)`
- 缩放变换：`scale(sx, sy)`

## 平移变换

顾名思义，就是一般的图形位移。例如我想把位于(100, 100)的矩形平移至(200, 200)点，那么只需要再绘制矩形之前加上`context.translate(100, 100)`即可，例子如下：

```javascript
let canvas = document.getElementById("canvas");
canvas.width = 800;
canvas.height = 600;

let context = canvas.getContext("2d");

context.fillStyle = "#00AAAA";
context.fillRect(100, 100, 200, 100);

context.fillStyle = "red";
context.translate(100, 100);
context.fillRect(100, 100, 200, 100);
```

如果想平移到(300, 300)的位置，需要重置参考坐标系的位置，也就是重置到原点，再进行平移，有两种方法：

- 绘制下一次平移图形的时候，手动将坐标系换回原点：即：`translate(-x, -y)`；
- 平移前调用`context.save()`和`context.restore()`。

实例如下：

```javascript
window.onload = function() {
    let canvas = document.getElementById("canvas");
    canvas.width = 800;
    canvas.height = 600;
    
    let context = canvas.getContext("2d");
    
    context.fillStyle = "#00AAAA";
    context.fillRect(100, 100, 200, 100);
    
    context.save();
    context.fillStyle = "red";
    context.translate(100, 100);
    context.fillRect(100, 100, 200, 100);
    context.restore();
    
    context.save();
    context.fillStyle = "green";
    context.translate(200, 200);
    context.fillRect(100, 100, 200, 100);
    context.restore();
};
```

## 旋转变换

`rotate(deg)`是指图形以坐标系原点为圆心进行顺时针旋转，因此在使用`rotate()`之前可以配合`translate()`来平移坐标系，确定旋转的圆心。

我们用一个例子来看看：

```javascript
// 画布
var canvas = document.getElementById("canvas");
canvas.width = 800;
canvas.height = 600;
// 画笔
var context = canvas.getContext("2d");

for (let i = 0; i <= 12; i++) {
    context.save();
    context.translate(70 + i * 50, 50 + i * 40);
    context.fillStyle = "#00AAAA";
    context.fillRect(0, 0, 20, 20);
    context.restore();

    context.save();
    context.translate(70 + i * 50, 50 + i * 40);
    context.rotate(i * 30 * PI / 180); // 这里是弧度，不是角度
    context.fillStyle = "red";
    context.fillRect(0, 0, 20, 20);
    context.restore();
}
```

## 缩放变换

缩放变换最简单了，分别是在水平方向和垂直方向上对象的缩放倍数。但是缩放会导致坐标位置改变、线条加粗。所以在实际使用过程中能尽量不使用scale函数做缩放变换是最好的。

```javascript
// 画布
var canvas = document.getElementById("canvas");
canvas.width = 800;
canvas.height = 600;
// 画笔
var context = canvas.getContext("2d");

context.strokeStyle = "red";
context.lineWidth = 5;
for (let i = 1; i < 4; i++) {
    context.save();
    context.scale(i, i);
    context.strokeRect(50, 50, 150, 100);
    context.restore();
}
```

## 万能的transform函数

前面讲的三种变换函数，使用`transform()`函数就可以做到，
