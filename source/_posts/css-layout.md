---
title: CSS 简单布局
date: 2019-09-26 08:32:30
tags: CSS
---

> CSS 布局元素包括漂浮 float，定位 position等；
> float 主要用于左右两边的漂浮，position 可用于整个文档的绝对定位，相对定位，固定漂浮位置等。

<!-- more -->

### 一，基础知识
##### 1. float : 让元素漂浮在文档的左边或者右边，默认值不漂浮
|值|描述|
|-|-|
|left|元素向左浮动。|
|right|元素向右浮动。|
|none|默认值。元素不浮动，并会显示在其在文本中出现的位置。|
|inherit|规定应该从父元素继承 float 属性的值。|

##### 2. position : 让元素定位于文档的某个位置，属性值更丰富
|值|描述|
|-|-|
|static|元素框正常生成。块级元素生成一个矩形框，作为文档流的一部分，行内元素则会创建一个或多个行框，置于其父元素中。|
|relative|元素框相对于第一个父节点偏移某个距离。元素仍保持其未定位前的形状，它原本所占的空间仍保留。|
|absolute|元素框从文档流完全删除，并相对于包含 position:relative/absolut 属性的第一个父节点来定位，不一定是直接的父节点。元素原先在正常文档流中所占的空间会关闭，就好像元素原来不存在一样。元素定位后生成一个块级框，而不论原来它在正常流中生成何种类型的框。|
|fixed|元素框的表现类似于将 position 设置为 absolute，不过其包含块是视窗本身。|



### 二，具体使用
##### 1. 实现左中右布局，要求左右固定宽度，中间自适应宽度
- float : [示例https://github.com/luckybirdme/blog/blob/master/example/css/float-for-layout.html)

``` html
<!DOCTYPE html>
<hmtl>
	<style type="text/css">
		/* 清除浏览器默认样式 */
		body{
			padding: 0 auto;
			margin: 0 auto;
		}
		/* 左边固定 */
		.left{
			float:left;
			width:100px;
			height:200px;
			background-color:green;
		}
		/* 右边固定 */
		.right{
			float:right;
			width:100px;
			height:200px;
			background-color:blue;
		}
		/* 中间自适应 */
		.middle{
			width:auto;
			height:auto;
			background-color: red;
		}

	</style>
<body >
	<div>
		<div class="left">left</div>
		<div class="right">right</div>
		<div class="middle">middle middle middle</div>
	</div>
</body>
</html>

```

- position : [示例https://github.com/luckybirdme/blog/blob/master/example/css/position-for-layout.html)

``` html
<!DOCTYPE html>
<hmtl>
	<style type="text/css">
		/* 清除浏览器默认样式 */
		body{
			padding: 0 auto;
			margin: 0 auto;
		}
		/* 左边固定 */
		.left{
			position: absolute;
			left: 0;
			top:0;
			width: 100px;
			height: 200px;
			background-color: green
		}
		/* 右边固定 */
		.right{
			position: absolute;
			right: 0;
			top:0;
			width: 100px;
			height: 200px;
			background-color: blue
		}
		/* 中间自适应 */
		.middle{
			margin: 0 100px;
			width: auto;
			height: auto;
			background-color: red
		}

	</style>
<body >
	<div>
		<div class="left">left</div>
		<div class="right">right</div>
		<div class="middle">middle middle middle</div>
	</div>
</body>
</html>

```


##### 2. 实现顶部菜单固定，右下角有返回顶部按钮
- postion : 利用fixed [示例](https://github.com/luckybirdme/blog/blob/master/example/css/position-fixed-for-layout.html)

``` html
<!DOCTYPE html>
<hmtl>
	<style type="text/css">
		/* 清除浏览器默认样式 */
		body{
			padding: 0 auto;
			margin: 0 auto;
		}
		/* 顶部固定 */
		.top{
			position: fixed;
			top:0;
			width: 100%;
			height: 50px;
			background-color: green
		}
		/* 右下角固定 */
		.back{
			position: fixed;
			bottom: 40px;
			right: 40px;
			width: 10px;
			height: 10px;
		}
		/* 中间自适应 */
		.middle{
			margin-top: 50px;
			background-color: red;
		}

	</style>
<body >
	<div>
		<div class="top">top</div>
		<div class="back">back</div>
		<div class="middle">middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle middle</div>
	</div>
</body>
</html>
```

##### 3. 实现水平居中
- 包括行内和块级元素 [示例](https://github.com/luckybirdme/blog/blob/master/example/css/float-middle-for-layout.html)

``` html
<!DOCTYPE html>
<hmtl>
	<style type="text/css">
		/* 清除浏览器默认样式 */
		body{
			padding: 0 auto;
			margin: 0 auto;
		}
		/* 行内元素居中 */
		.line-middle{
			text-align: center;
		}
		/* 确定宽度块内元素居中 */
		.block-middle{
			width: 100px;
			position: absolute;
			left: 50%;
			margin-left: -50px; /* 修复固定宽度的偏差值 */
		}
	</style>
<body >
	<div class="line-middle">
		<span>line-middle</span>
	</div>
	<div class="block-middle">
		<div class="block-middle">
			block-middle block-middle block-middle block-middle block-middle block-middle
		</div>
	</div>
</body>
</html>
```

##### 3. 实现水平和垂直居中
-  两个方向都居中 [示例](https://github.com/luckybirdme/blog/blob/master/example/css/float-middle-for-layout.html)

``` html
<!DOCTYPE html>
<html>
<head>
    <style type="text/css">
        /* 设置垂直满屏显示，方便居中 */
        html,body{
            height: 100%;
            width: 100%;
        }
        /* 清除浏览器默认样式 */
        body{
            padding: 0 auto;
            margin: 0 auto;
        }
        /* 水平和垂直居中 */
        .content {
            width: 300px;
            height: 300px;
            background: red;
            margin: 0 auto; /*水平居中*/
            position: relative;
            top: 50%; /*垂直居中*/
            /*修复垂直居中的偏移*/
            margin-top: -150px; 
            /* transform: translateY(-50%);/* css3 */
        }
    </style>
</head>
<body>
    <div class="content"></div>
</body>
</html>
```