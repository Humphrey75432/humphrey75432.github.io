---
title: Canvas非零环绕原则
date: 2022-02-18 10:59:52
updated: 2022-02-18 10:59:52
tags: HTML
categories: 前端
---

# 非零环绕原则

## 概述

非零环绕原则描述了一种计算机该如何给封闭图形上色的规则，非常简单，我们只需要记住一个下面一条规则就行。

给封闭图形设置一个渲染方向（假如你的图形是一笔成画的），比如顺时针，那么与之相反的逆时针就是反方向。根据线条的方向来按照下列规则判断：

- 从封闭图形画一条射线，如果与之相交的线条中正向和逆向不等（正多逆少或者逆多正少，只要不是0即可），那么就认为图形内部；
- 从封闭图形画一条射线，如果与之相交的线条中正向和逆向相等（例如圆环，一正一逆，正好抵消为0），那么就认为图形外部；

这就是“非零环绕原则”。这样说还不直观，我们可以用一个例子来说明这个。

## 圆环

接着之前的代码片段，我们来画一个圆环：

```javascript
window.onload = function() {
  let canvas = document.getElementById("canvas");
  canvas.width = 800;
  canvas.height = 600;
  
  let context = canvas.getContext("2d");
  
  context.shadowColor = "#545454";
  context.shadowOffsetX = 5;
  context.shadowOffsetY = 5;
  context.shadowBlur = 2;
  
  context.arc(400, 300, 200, 0, Math.PI * 2, false); // 顺时针画圆
  context.arc(400, 300, 230, 0, Math.PI * 2, true); // 逆时针画圆
  
  context.fillStyle = "#00AAAA";
  context.fill();
};
```

 按照“非零环绕”原则，内部圆的方向是顺时针，外部圆的方向是逆时针，正好两个圆的中间部分才是需要填充的部分。这样就完成了一个圆环。是不是非常简单。

## 镂空图形绘制

我们再做一个复杂一些的图形，你可以将其拷贝到浏览器中看效果：

```javascript
window.onload = function() {
  var canvas = document.getElementById("canvas");
  canvas.width = 800;
  canvas.height = 600;
  var context = canvas.getContext("2d");
  context.fillStyle = "#FFF";
  context.fillRect(0,0,800,600);

  context.beginPath();
  context.rect(200,100,400,400);
  drawPathRect(context, 250, 150, 300, 150);
  drawPathTriangle(context, 345, 350, 420, 450, 270, 450);
  context.arc(500, 400, 50, 0, Math.PI * 2, true);
  context.closePath();

  context.fillStyle = "#058";
  context.shadowColor = "gray";
  context.shadowOffsetX = 10;
  context.shadowOffsetY = 10;
  context.shadowBlur = 10;
  context.fill();

};

//逆时针绘制矩形
function drawPathRect(cxt, x, y, w, h){
  /**
         * 这里不能使用beginPath和closePath，
         * 不然就不属于子路径而是另一个全新的路径，
         * 无法使用非零环绕原则
         */
  cxt.moveTo(x, y);
  cxt.lineTo(x, y + h);
  cxt.lineTo(x + w, y + h);
  cxt.lineTo(x + w, y);
  cxt.lineTo(x, y);

}

//逆时针绘制三角形
function drawPathTriangle(cxt, x1, y1, x2, y2, x3, y3){
  cxt.moveTo(x1,y1);
  cxt.lineTo(x3,y3);
  cxt.lineTo(x2,y2);
  cxt.lineTo(x1,y1);
}
```

## 结束

就这些
