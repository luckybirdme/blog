---
title: 伪类和伪元素
date: 2019-06-11 07:36:25
tags: CSS
---

> 伪类和伪元素

<!-- more -->


## 一. 伪类(pseudo-class)
1. 作用
为了通过选择器，格式化DOM树以外的信息以及不能被常规CSS选择器获取到的信息。
2. 场景

- 格式化DOM树以外的信息。比如： a 标签的:visited,以及 input 标签的 :focus, 根据状态而改变的样式，这些信息不存在于DOM树中。
- 不能被常规CSS选择器获取到的信息。比如：要获取第一个子元素，我们无法用常规的CSS选择器获取，但可以通过 :first-child 来获取到。


## 二. 伪元素(Pseudo-elements )
1. 作用
可以创建一些文档语言无法创建的虚拟元素。

2. 场景

- 文档语言没有一种机制可以描述元素内容的第一个字母或第一行，但伪元素可以做到(::first-letter,::first-line)。
- 伪元素还可以创建源文档不存在的内容，比如在元素前后使用 ::before 或 ::after。

## 三. 区别
1. 操作对象 

- 伪类其实是弥补了CSS选择器的不足，用来更方便地获取信息。
- 伪元素本质上是创建了一个虚拟容器(元素)，是不存在的元素。

2. 使用方式

- 在CSS3中，伪类用单冒号:表示，伪元素用双冒号::表示
- 为了兼容，目前依然支持单冒号:表示伪类，::before和:before效果一样


## 四. 示例

```html
<!DOCTYPE html>
<html>

    <head>
        <meta charset="utf-8" />
        <title>测试案例</title>
        <style type="text/css">
            /* 普通样式 */
            .first_letter {
				color:blue;
				font-size:20px;
			}
			/* 伪元素样式 */
			#test_1 > p::first-letter {
				color:red;
				font-size:30px;
			}
			#test_2 > p::before { 
			  content: "«";
			  color: red;
			  margin:auto 10px;
			}
			#test_2 > p::after { 
			  content: "»";
			  color: red;
			  margin:auto 10px;
			}
			

            /* 普通样式 */
			li.first-item {
				color: blue;
				font-size:20px;
			}
			/* 伪类 */
			#test_3 > li:first-child {
				color: red;
				font-size:30px;
			}
			a:hover {
			  color: red;
			  font-size:30px;
			}
						
        </style>
    </head>

    <body>
        <div>
			<div>
				<p><span class="first_letter">H</span>ello World</p>
			</div>
			<div id="test_1">
				<p>Hello World</p>
			</div>
			<div id="test_2">
				<p>Hello World</p>
			</div>
		</div>
		<div>
			<div>
				<ul>
					<li class="first-item">我是第一个</li>
					<li>我是第二个</li>
				</ul>
			</div>
			<div>
				<ul id="test_3">
					<li>111</li>
					<li>222</li>
				</ul>  
			</div>
			<div>
				<a href="" > a link </a>
			</div>
		</div>
    </body>

</html>
```