---
title: CSS3 动画效果
date: 2019-09-26 15:57:23
tags: CSS
---

> CSS3 新增了动画效果，通过 keyframes 和 animation 来实现；
> 由于各大浏览器实现方式不一样，需要考虑兼容性，而且 IE 版本 10 以上才支持。

<!-- more -->

### 一，基础知识
##### 1. keyframes 定义具体的动画效果
- 注意浏览器兼容性

```css
@keyframes myfirst
{
	from {background: red;}
	to {background: yellow;}
}

/* Firefox */
@-moz-keyframes myfirst 
{
	from {background: red;}
	to {background: yellow;}
}

/* Safari and Chrome */
@-webkit-keyframes myfirst
{
	from {background: red;}
	to {background: yellow;}
}

/* Opera */
@-o-keyframes myfirst
{
	from {background: red;}
	to {background: yellow;}
}
```
##### 2. animation 绑定具体的动画元素
- 注意浏览器兼容性

```css
/* 绑定元素 */
.one-animation {
	width:100px;
	height:100px;
	background:red;
	float: left;
	animation:myfirst 5s;
	-moz-animation:myfirst 5s; /* Firefox */
	-webkit-animation:myfirst 5s; /* Safari and Chrome */
	-o-animation:myfirst 5s; /* Opera */
}
```

### 二，具体使用
##### 1. 颜色变化和位置移动，[示例](/example/css/keyframes-animation.html)