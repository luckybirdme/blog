---
title: canvas 和 svg
date: 2019-10-30 07:46:03
tags: html
---
> svg, 基于 xml 来描述矢量图形，好比 html 来描述文本一样。
> canvas, 通过 JS 来绘制位图，达到各种 2D 图片或动画效果。

<!-- more -->

### 一，svg
##### 1.原理：基于 xml 来描述矢量图形，好比 html 来描述文本一样，比如通过 circle 元素描述圆形
##### 2.基本使用
```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
    <svg version="1.1"
         baseProfile="full"
         width="200" height="200"
         xmlns="http://www.w3.org/2000/svg">

      <rect width="100%" height="100%" fill="red" />

      <circle cx="100" cy="100" r="80" fill="green" />

      <text x="100" y="125" font-size="60" text-anchor="middle" fill="white">TEST</text>

    </svg>
</body>
</html>
```


<iframe
    frameborder="0" scrolling="0" width="320" height="220"
    src="/example/js/svg.html" >
</iframe>



### 二，canvas
##### 1.原理：通过 JS 来绘制位图，达到各种 2D 图片或动画效果
##### 2.基本使用
```html
<!DOCTYPE html>
<html>
 <head>
  <meta charset="utf-8"/>
  <script type="application/javascript">
    function draw() {
      var canvas = document.getElementById('canvas');
      if (canvas.getContext) {
        var ctx = canvas.getContext('2d');

        ctx.fillStyle = 'rgb(200, 0, 0)';
        ctx.fillRect(10, 10, 50, 50);

        ctx.fillStyle = 'rgba(0, 0, 200, 0.5)';
        ctx.fillRect(30, 30, 50, 50);
      }
    }
  </script>
 </head>
 <body onload="draw();">
   <canvas id="canvas" width="150" height="150"></canvas>
 </body>
</html>
```

<iframe
    frameborder="0" scrolling="0" width="320" height="220"
    src="/example/js/canvas.html" >
</iframe>


### 三，两者应用场景的差异

|分类|svg|canvas|
|-|-|-|
|历史由来|绘制矢量图形的 xml 语言，2003年成为 W3C 的标准|使用脚本 (JS) 来绘制位图，由 Apple 引入|
|构图元素|多种图形元素，text,circle,pattern等|只有 canvas 节点 |
|成像原理|保存图形对象信息，以矢量图展示，放大不会失真|将所有图形合并，以位图方式保存和展示，放大时会失真|
|绘制图形|支持 CSS 和脚本 | 仅限脚本 JavaScript|
|事件绑定|支持图形任一元素 (text,circle) 的事件绑定|只有 canvas 一个节点，图形中元素无法绑定事件|
|渲染性能|每修改一个图形元素都需要重新渲染|通过离屏 canvas 缓存图形，提高渲染效率|
|应用场景|大型渲染区的应用，每个元素有响应事件，比如地图|图像密集型的应用，可利用缓存图形，比如游戏|

### 四，成熟前端图形展示框架
##### 1. 基于 svg 的 [highcharts](https://www.highcharts.com)
##### 2. 基于 canvas 的 [chartjs](https://www.chartjs.org/)
