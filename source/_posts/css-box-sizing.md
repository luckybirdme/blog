---
title: 盒子模型
date: 2019-06-10 07:30:06
tags: CSS
---

> 盒子模型

<!-- more -->


## 一. 概念
CSS盒模型本质上是一个盒子，封装周围的HTML元素，它包括：外边距（margin）、边框（border）、内边距（padding）、实际内容（content）四个属性。

## 二. 类型
目前存在两种CSS盒模型

##### 1. W3C盒子模型(标准盒模型)
盒子总宽度/高度 = width/height + padding + border + margin。
即 width/height 只是内容高度，不包含 padding 和 border 值 
属性 box-sizing: content-box; 浏览器默认设置

![](/img/2020/box_sizing_1.jpg)



##### 2. IE盒子模型(怪异盒模型)
盒子总宽度/高度 = width/height + margin = (内容区宽度/高度 + padding + border) + margin。
即 width/height 包含了 padding 和 border 值 
属性 box-sizing: border-box; IE 默认属性

![](/img/2020/box_sizing_2.jpg)


##### 特别说明：
- box-sizing:inherit , 规定应从父元素继承 box-sizing 属性的值
- 盒子模型主要针对块级元素，即 display:block；
- 对于行内元素，必须设置成 display:inline-block; 即行内块元素，才能利用盒子模型的属性。


## 三. 特点

##### 1. 两个块级元素，上下的外边距会被合并，合并后的外边距的高度等于两个发生合并的外边距中较高的那个外边距值。需要注意的是：只有普通文档流中块框的垂直外边距才会发生外边距合并。行内快级元素、浮动框 float 、绝对定位 position 之间的外边距不会合并。


![](/img/2020/box_sizing_3.jpg)


##### 3. 块级元素是矩形，当边框 border 有宽度且相交时，会以对角线划分，再借助 transparent 透明颜色的特性，可以画出有颜色的三角形，或者有边框颜色的矩形，菱形

```css
.box1{
  width:50px;
  height:50px;
  background:yellow;
  border-top: 20px solid red;
  border-right:20px solid black;
  border-bottom:20px solid green;
  border-left:20px solid blue;
}
.box2{
  width:10px;
  height:10px;
  background:yellow;
  border-top: 20px solid red;
  border-right:20px solid black;
  border-bottom:20px solid green;
  border-left:20px solid blue;
}
.box3{
  width:0px;
  height:0px;
  background:yellow;
  border-top: 20px solid red;
  border-right:20px solid black;
  border-bottom:20px solid green;
  border-left:20px solid blue;
}

.box4{
  width:0px;
  height:0px;
  border-top: 20px solid red;
  border-right:20px solid transparent;
  border-left:20px solid transparent;
}
```

![](/img/2020/box_sizing_5.jpg)


##### 3. 浮动框 float ，绝对定位 position，以及 CSS3 的 flex box 弹性盒子会改变块级元素的展示方式，也就会影响盒子模型的展示。


```css
.box {
  display: inline-block;
  width: 100px;
  height: 100px;
  background: red;
  color: white;
}

#two {
  position: relative;
  top: 20px;
  left: 20px;
  background: blue;
}
```

![](/img/2020/box_sizing_4.jpg)

