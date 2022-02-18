---
title: Canvas绘制图形
date: 2022-02-18 10:59:22
updated: 2022-02-18 10:59:22
tags: HTML
categories: 前端
---

# Canvas绘图

## 回顾

前一章讲了如何创建绘图的几个基本要素，它们分别是：

- 画布
- 画笔
- 颜色
- 橡皮擦（这个现在还用不着）

光有这些还不够，一幅画主要由线条，形状（圆形或者矩形），弧线组成，所以要学会用代码画图，那么得先知道如何在电脑上画出这些线条。我们直接拿之前的HTML 5页面来使用：

```html
<!DOCTYPE html>
<html lang="zh">
<head>
	<title>Canvas Tutorial</title>
</head>
<body>
	<div id="canvas-wrap">
		<canvas id="canvas" style="border: 1px solid #aaaaaa; display: block; margin: 50px auto;">
			抱歉你的浏览器不支持Canvas!
		</canvas>
	</div>
</body>

<script type="text/javascript">
	// 创建画布
    let canvas = document.getElementById("canvas");
    canvas.width = 800;
    canvas.height = 600;
    // 创建画笔
    let context = canvas.getContext("2d");
</script>
</html>
```

好了，接下来，我们将开始学习如何画出线条、矩形、圆形和弧形；

## 线条

线条比较简单，中学的时候我们都知道，两点就能确定一条直线，所以在计算机中也是一样，只要我给定两个点的坐标，那么他们之间的轨迹就是一条直线。所以直线的绘制方法如下：

```javascript
let canvas = document.getElementById("canvas");
canvas.width = 800;
canvas.height = 600;

let context = canvas.getContext("2d");

context.beginPath();
context.moveTo(100, 100);
context.lineTo(500, 100);
context.stroke();
```

其中`moveTo(x, y)`代表了直线的绘制起点，`lineTo(x, y)`代表了直线的绘制终点；

由于canvas是基于状态绘制的，所以每绘制一个图形，都要以`context.beginPath()`起头，如果要上色，可以使用`context.strokeStyle = <color>`给线条上色。

这里的坐标用的同样是笛卡尔坐标系，但是跟我们中学课本上的不一样。在计算机中，屏幕的左上角是坐标顶点，横向是x正轴，纵向是y正轴；

## 矩形

要绘制矩形，可以使用两种办法：

（1）使用之前的`moveTo`和`lineTo`方法，一次性绘制4条直线，那自然就围城了一个矩形；

（2）直接调用canvas封装的API：`rect(x, y, width, height)`来绘制矩形；

这两种方法都可以在页面上绘制出一个矩形，但是使用方法（1）的时候必须要在绘制结束后加上`context.closePath()`，否则图形看上去就会有缺口；

你可以把下面这两段直接拷贝到自己的代码中，就可以看到效果了：

```javascript
let canvas = document.getElementById("canvas");
let context = canvas.getContext("2d");

// 方法（1）绘制矩形
context.beginPath();
context.moveTo(100, 100);
context.lineTo(400, 100);
context.lineTo(400, 400);
context.lineTo(100, 400);
context.closePath(); // 要想图形闭合，需要加上这句话
context.strokeStyle = "blue";
context.stroke();

// 方法(2)绘制矩形
context.beginPath();
context.rect(100, 100, 300, 300);
context.strokeStyle = "red";
context.stroke();
```

最后，我们再说一下线条属性：

- lineCap：定义上下文中线的端点样式：可以是：butt、round和square；
- lineJoin：定义两条线相交产生的拐角，可以称为连接，可以是：miter、bevel和round；
- lindWidth：定义线条宽度，默认是1.0；
- strokeStyle：定义线和形状边框的颜色样式；

## 弧线

熟悉PS的同学也会知道，绘制圆弧常用的工具有预设的圆弧工具和钢笔工具（贝塞尔曲线）；那么在canvas中也是一样的：

- 标准圆弧：`arc()`
- 复杂圆弧：`arcTo()`
- 2阶贝塞尔曲线：`quadraticCurveTo()`
- 3阶贝塞尔曲线：`bezierCurveTo()`

### `context.arc(x, y, radius, startAngle, endAngle, antiClockWise)`

参数代表的含义分别为：圆心坐标，圆心半径，起始弧度，结束弧度以及是否逆时针；

这里的弧度和中学课本里面讲的弧度是同一个概念，只不过坐标系的参考位置是不一样的；

### `context.arcTo(x1, y1, x2, y2, radius)`

参数代表的含义分别为：两个切点的坐标和圆弧半径。这个方法是依据切线来画弧线，即两条切线就能确定一条弧线；例子如下：

```javascript
let canvas = document.getElementById("canvas");
canvas.width = 800;
canvas.height = 600;

let context = canvas.getContext("2d");

context.beginPath();
context.moveTo(200, 200);
context.arcTo(600, 200, 600, 400, 100);

context.lineWidth = 6;
context.strokeStyle = "red";
context.stroke();

context.beginPath();
context.moveTo(200, 200);
context.lineTo(600, 200);
context.lineTo(600, 400);

context.lineWidth = 1;
context.strokeStyle = "#0088AA";
context.stroke();
```

## 圆形

有了上面绘制圆弧的基础知识，我们就可以绘制圆形了，其实也非常简单。

```javascript
const PI = Math.PI;
let canvas = document.getElementById("canvas");
canvas.width = 800;
canvas.height = 600;

let context = canvas.getContext("2d");

context.beginPath();
context.arc(200, 200, 50, 0, PI * 2); // 顺时针绘制
context.arc(400, 400, 50, 0, PI * 2, true); // 逆时针绘制
```

记住计算机上圆形的弧度顺序：时钟3点对应的是0(PI * 2)，时钟6点对应的是PI / 2，时钟9点对应的是PI，时钟12点对应的是PI * 3 / 2；默认情况下绘图顺序是顺时针方向；

## 曲线

如果你用过PS的钢笔工具，那么一定也画过曲线，钢笔工具画出来的曲线就叫做贝塞尔曲线，这是发过数学家贝塞尔于1962年发现，并以他的名字命名了这种曲线。

曲线由起始点、终止点和控制点组成，每条曲线都具备这三个点。控制点为曲线阶数 - 1，也就是说2阶贝塞尔曲线有2 - 1 = 1个控制点。

### 2阶贝塞尔曲线

canvas中，绘制二阶贝塞尔曲线的API如下：

```javascript
context.quadraticCurveTo(cpx, cpy, x, y); // 2阶
```

一个控制点（cpx, cpy）和一个终止点（x, y）；

### 3阶贝塞尔曲线

绘制三阶贝塞尔曲线的API如下：

```javascript
context.bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y); // 3阶
```

两个控制点（cp1x, cp1y）和（cp2x, cp2y）和终止点（x, y）
