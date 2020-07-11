---
title: 块级和行内元素
date: 2019-06-10 07:55:21
tags: CSS
---

> 块级和行内元素

<!-- more -->


## 一. 分类区别

分类|块级元素|行内元素|行内块级元素
-|-|-|-
padding| 上下左右有效|上下左右有效|上下左右有效
margin|上下左右有效|左右有效，上下无效|上下左右有效
width|有效|无效|有效
height|有效|无效|有效
文字换行|自动换行|不换行|不换行
属性切换|display:block;|display:inline;|display:inline-block;

## 二. 示例代码

- 块级元素

```html
<!DOCTYPE html>
<html>

    <head>
        <meta charset="utf-8" />
        <title>测试案例</title>
        <style type="text/css">
            div {
                width: 120px;
                height: 120px;
                margin: 50px 50px;
                padding: 50px 40px;
                background: lightblue;
            }
        </style>
    </head>

    <body>
        <i>自动换行</i>
        <div>块状元素</div>
        <div>块状元素</div>
    </body>

</html>
```


- 行内元素

```html
<!DOCTYPE html>
<html>

    <head>
        <meta charset="utf-8" />
        <title>测试案例</title>
        <style type="text/css">
            span {
                width: 120px;
                height: 120px;
                margin: 1000px 20px;
                padding: 50px 40px;
                background: lightblue;
            }
        </style>
    </head>

    <body>
        <i>不会自动换行</i>
        <span>行内元素</span>
    </body>

</html>

```


- 行内块级元素

```html
<!DOCTYPE html>
<html>

    <head>
        <meta charset="utf-8" />
        <title>测试案例</title>
        <style type="text/css">
            div {
                display: inline-block;
                width: 100px;
                height: 50px;
                margin: 20px 20px;
                padding: 50px 40px;
                background: lightblue;
            }
        </style>
    </head>

    <body>
        <div>行内块状元素</div>
        <div>行内块状元素</div>

    </body>

</html>
```


