---
title: CSS 弹性盒子
date: 2019-09-26 11:16:26
tags: CSS
---

> 弹性盒子 flex box 是 CSS3 的新特性，是为了方便在不同屏幕大小展示布局，也为了更高效地对元素进行排列

<!-- more -->

### 一，基础知识
##### 1. 弹性盒子 flex box 是 CSS3 的新特性，是为了方便在不同屏幕大小展示布局，也为了更高效地对元素进行排列
##### 2. 使用 flex box 时，首先定义一个 dom 节点，添加 flex 属性以及相关属性，注意浏览器兼容性问题
- 定义 flex box 

```css
/* flex 弹性布局 */
.flex-box {
    /* 浏览器兼容性 */
    display: -webkit-box;
    display: -ms-flexbox;
    display: -webkit-flex;
    display: flex;
}
```

##### 3. 定义了 flex 的 dom 的相关属性有
- flex-direction : 排列方式，[示例](/example/css/flex-box-for-layout.html)
|值|描述|
|-|-|
row|横向从左到右排列（左对齐），默认的排列方式。
row-reverse|反转横向排列（右对齐，从后往前排，最后一项排在最前面。
column|纵向排列。
column-reverse|反转纵向排列，从后往前排，最后一项排在最上面。



- flex-wrap : 换行方式，[示例](/example/css/flex-box-wrap-for-layout.html)
|值|描述|
|-|-|
nowrap|默认值，不换行。
wrap|主轴宽度超过限定时，自动换行，第一行在上方。
wrap-reverse|换行，第一行在下方。

- flex-flow : flex-direction属性和flex-wrap属性的简写形式 

```css
flex-direction : row;
flex-wrap : nowrap;
/* 相当于简写模式 */
flex-flow : row nowrap;
```

- justify-content : 水平方向的对齐方式，[示例](/example/css/flex-box-justify-content-for-layout.html)
|值|描述|
|-|-|
flex-start|默认值。项目位于容器的开头。
flex-end|项目位于容器的结尾。
center|项目位于容器的中心。
space-between|项目位于各行之间留有空白的容器内。
space-around|项目位于各行之前、之间、之后都留有空白的容器内。

- align-items : 垂直方向的对齐方式，[示例](/example/css/flex-box-align-items-for-layout.html)
|值|描述|
|-|-|
stretch|默认值。项目被拉伸以适应容器。
center|项目位于容器的中心。
flex-start|项目位于容器的开头。
flex-end|项目位于容器的结尾。
baseline|项目位于容器的基线上。

- align-content : align-items 是垂直方向一条轴的对齐方式，align-content 则是多条轴的垂直对齐方式
|值|描述|
|-|-|
flex-start|与交叉轴的起点对齐。
flex-end|与交叉轴的终点对齐。
center|与交叉轴的中点对齐。
space-between|与交叉轴两端对齐，轴线之间的间隔平均分布。
space-around|每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
stretch|默认值，轴线占满整个交叉轴。

##### 4. flex 的 dom 的下子节点相关属性，[示例](/example/css/flex-item-for-layout.html)
- order : 整数(0,1,2,3,4)，排序的权重
- flex-grow : 整数(0,1,2,3,4)，宽度放大比例
- flex-shrink : 整数(1,2,3,4)，宽度缩小比例
- flex-basis : 属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小
- flex : 是flex-grow, flex-shrink 和 flex-basis的简写，默认值为 0 1 auto
- align-self : 属性允许单个项目有与其他项目不一样的垂直对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch


### 二，具体使用
##### 1. 水平和垂直居中
- 块级元素水平和垂直居中 [示例](/example/css/flex-middle-for-layout.html)

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
            padding: 0 ;
            margin: 0 ;
        }
        /* flex 弹性布局 */
        .flex-box {
            /* 浏览器兼容性 */
            display: -webkit-box;
            display: -ms-flexbox;
            display: -webkit-flex;
            display: flex;
            height: 100%;
            width: 100%;
        }
        .box{
            background-color: red;
            width: 80px;
            margin: 5px;
            text-align: center;
        }
        .middle{
            /* 垂直对齐 */
            align-items: center;
            /* 水平对齐 */
            justify-content: center;
        }
    </style>
</head>
<body>
    <div class="flex-box middle" >
        <span class="box">box 1</span>
        <span class="box">box 2</span>
    </div>
</body>
</html>
```

- 登录框垂直居中，并且输入框对齐 [示例](/example/css/flex-input-for-layout.html)

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
            padding: 0 ;
            margin: 0 ;
        }
        /* flex 弹性布局 */
        .flex-box {
            /* 浏览器兼容性 */
            display: -webkit-box;
            display: -ms-flexbox;
            display: -webkit-flex;
            display: flex;
            /* 设置垂直满屏显示，方便居中 */
            height: 100%;
            width: 100%;
            /* 垂直居中 */
            align-items: center;
            /* 水平居中 */
            justify-content: center;
            /* 分行显示 */
            flex-direction: column;
        }
        .flex-box-two{
            /* 浏览器兼容性 */
            display: -webkit-box;
            display: -ms-flexbox;
            display: -webkit-flex;
            display: flex;
            width: 300px;
            /* 右边对齐 */
            justify-content: flex-end;
            margin: 10px auto;
        }
        .input-class{
            width: 120px;
            margin-left: 10px;
        }
    </style>
</head>
<body>
    <div class="flex-box" >
        <span class="flex-box-two">
            <label>name : </label>
            <input type="text" name="name" class="input-class">
        </span>
        <span class="flex-box-two">
            <label>password : </label>
            <input type="password" name="password" class="input-class">
        </span>
        <span class="flex-box-two">
            <label>confirm password : </label>
            <input type="password" name="password" class="input-class">
        </span>
    </div>
</body>
</html>
```