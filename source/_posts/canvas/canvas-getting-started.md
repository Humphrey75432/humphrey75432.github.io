---
title: Canvas基础入门
date: 2022-02-18 10:59:03
updated: 2022-02-18 10:59:03
tags: HTML
categories: 前端
---

# Canvas快速上手

## 准备工作

你只需要准备：

- Web Kit内核浏览器：Safari（Apple MacOS）、Google Chrome或者Mozilla Firefox；（由于Canvas的标准是Chrome和Mozilla联合制定的，所以推荐使用Chrome或者Firefox）
- 你最拿手的编辑器：Sublime Text、Atom或者是Visual Studio Code；（这里我选择了VS Code）

准备好上面这些之后，在你的电脑文件系统中找一个位置，然后创建一个HTML 5的页面，像下面这样：

```html
<!DOCTYPE html>
<html lang="zh">
<head>
	<title>Canvas Tutorial</title>
</head>
<body>
</body>
</html>
```

准备好页面以后，我们在`<body></body>`标签中写一点东西，来验证一下`<canvas>`的魅力，像下面这样：

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
</html>
```

可以发现，当你打开浏览器以后，你的页面上什么都看不到，这恰恰说明你的浏览器是支持Canvas的！你可以找一个老古董浏览器（例如：Internet Explorer）来打开看看，它就会显示“<u>抱歉你的浏览器不支持Canvas！</u>”

## 计算机绘图基础

现实生活中，要想绘图，你需要准备下面这些东西：

- 画布（当然不一定是真的布，纸张也算是画布的一种）
- 画笔（各种粗细不一，功能不同的笔）
- 颜料（12色、16色或者24色的颜料板）
- 橡皮、尺子以及其他辅助工具等等；

那么将这些东西映射到计算机上呢？答案也是一样的，你同样需要准备上面的那几样东西，只不过我们把现实生活中的工具都转换成了JavaScript代码而已。

### 画布

我们再次看`<canvas></canvas>`标签，它有一个id属性，这个id所指向的DOM元素就是我们的画布了。

```html
<canvas id="canvas" style="border: 1px solid #aaaaaa; display: block; margin: 50px auto;">
			抱歉你的浏览器不支持Canvas!
		</canvas>
```

我们可以给画布添加一些CSS样式，这样在浏览器中你就能看到画布的位置了：

```css
#canvas {
    border: 1px solid #aaaaaa;
    display: block;
    margin: 50px auto;
    width: 800;
    height: 600;
}
```

用JavaScript也可以设置画布的宽和高，像下面这样：

```javascript
window.onload = function() {
    var canvas = document.getElementById("canvas"); // 这样就可以取到画布对象了
    // 设置画布的大小
    canvas.width = 800;
    canvas.height = 600;
};
```

到此画布我们就创建好了，今后都会在画布上创建图形。

### 画笔

有了画布，自然就少不了画笔了，一般在其他教程中都会叫做上下文环境，我觉得这个词过于抽象，还是用画笔比较容易理解。

```javascript
window.onload = function () {
    // 画布
    var canvas = document.getElementById("canvas");
    canvas.width = 800;
    canvas.height = 600;
    
    // 画笔
    var context = canvas.getContext("2d");
};
```

那么既然是画笔，自然就可以设置它的粗细，颜色了对吧，说得对，确实是可以设置：

```javascript
var context = canvas.getContext("2d");
context.lineWidth = 5; //画笔的粗细
context.strokeStyle = "#acef4d"; // 画笔的颜色
```

### 颜料

说完画布和画笔，我们来说颜料；计算机中的颜色表示相信大家都知道了。你可以用`RGB`或者`HSLA`颜色表示法，也可以像学习CSS那样给图形上颜色；非常简单。

上色的方法也非常简单，只需要再创建画笔之后使用`context.fillStyle`就可以了。上色的方式支持下面这些：

- 颜色字符串，例如：red，blue和purple；
- 使用十六进制字符串或者其简写形式填充：#FF0000或者#FF0;
- 使用`rgb()`方法设置颜色；R红色、G绿色、B蓝色；
- 使用`rgba()`方法设置颜色；A代表(Alpha)，代表透明度；（0 ~ 1）；
- 使用`hsl()`方法设置颜色；H色相、S饱和度、L明度；
- 使用`hsla()`方法设置颜色；A代表透明度(Alpha)；（0 ~ 1）；

### 渐变效果

渐变分为两种，熟悉PS的同学应该知道是什么；

- 线性渐变：`context.createLinearGradient(xstart, ystart, xend, yend)`
- 径向渐变：`context.createRadialGradient(x0, y0, r0, x1, y1, r1)`

添加渐变的步骤有下面3个：

（1）添加渐变线

（2）为渐变线添加关建色；

（3）应用渐变；

转换成代码就是下面这样子：（以线性渐变为例）

```javascript
// 添加渐变线
let grd = context.createLinearGradient(xstart, ystart, xend, yend);
let grd2 = context.createRadialGradient(x0, y0, r0, x1, y1, r1);

// 添加关建色
/* stop代表到(xstart, ystart)的距离占整个渐变色长度的比例，为0~1的浮点数 */
grd.addColorStop(stop, color)

// 应用渐变
context.fillStyle = grd; // 填充渐变色
context.strokeStyle = grd; // 线框渐变色
```

贴上一个例子方便理解：

```javascript
// Code should wrap in HTML script tag
window.onload = function() {
    let canvas = document.getElementById("canvas");
    let context = canvas.getContext("2d");
    
    let grd = context.createRadialGradient(400, 300, 100, 400, 300, 200);
    
    grd.addColorStop(0, "olive");
    grd.addColorStop(0.25, "maroon");
    grd.addColorStop(0.5, "aqua");
    grd.addColorStop(0.75, "fuchsia");
    grd.addColorStop(0.25, "teal");
    
    context.fillStyle = grd;
    
    context.fillRect(100, 100, 600, 400);
};
```

### 橡皮擦

canvas提供了一个API叫做`context.clearRect(x, y, w, h)`，这个API就是用来清除画布中的像素的，也就是现实作画中的橡皮擦。
