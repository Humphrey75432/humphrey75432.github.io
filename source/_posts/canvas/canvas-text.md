---
title: Canvas文本
date: 2022-02-18 11:00:01
updated: 2022-02-18 11:00:01
tags: HTML
categories: 前端
---

# Canvas文本

## 文本API

使用Canvas显示字体分下面三步：

- 使用`font`设置字体；
- 使用`fillStyle`设置字体颜色；
- 使用`fillText()`方法显示字体；

默认情况下`font`属性可以不指定，如果不指定字体，则默认使用10px无衬字体；贴上一个例子：

```javascript
window.onload = function() {
    let canvas = document.getElementById("canvas");
    canvas.width = 800;
    canvas.height = 600;
    
    let context = canvas.getContext("2d");
    
    context.font = "50px serif";
    context.fillStyle = "#00AAAA";
    context.fillText("Canvas -- Draw on the web", 50, 300);
};
```

## 文本渲染

和图形一样，文本也提供了`fillText()`和`strokeText()`两种方法。具体看例子：

```javascript
window.onload = function() {
    let canvas = document.getElementById("canvas");
    canvas.width = 800;
    canvas.height = 600;
    
    let context = canvas.getContext("2d");
    
    context.beginPath();
    context.font = "50px Verdana";
    var gradient = context.createLinearGradient(0, 0, 800, 0);
    gradient.addColorStop("0", "magenta");
    gradient.addColorStop("0.5", "blue");
    gradient.addColorStop("1.0", "red");
    
    context.fillStyle = gradient;
    context.strokeStyle = "#00AAAA";
    context.strokeText("Text", 50, 100);
    context.fillText("Text", 50, 100);
    
    context.fillText("Text", 50, 300, 200);
};
```
