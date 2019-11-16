---
title: 浏览器页面呈现过程
date: 2019-11-13 07:52:16
tags: html
---

> 浏览器通过 http 协议获取到页面内容，然后进行解析，渲染，布局，绘制等步骤，最终将页面呈现给用户。
> 页面呈现过程会因为一些特殊步骤而阻塞，比如下载外部资源，执行 JS 等。

<!-- more -->

## 一，页面呈现过程
---

##### 浏览器通过 http 协议获取到页面内容，然后进行解析，渲染，布局，绘制等步骤，最终将页面呈现给用户，如下图所示

![](http://qiniucdn.luckybird.me/blog/img/2019/browser-page-build.png)


#### 1. 解析 html，生成 DOM

![](http://qiniucdn.luckybird.me/blog/img/2019/html-dom.png)


#### 2. 解析 CSS，生成 CSSOM

![](http://qiniucdn.luckybird.me/blog/img/2019/css-dom.png)


#### 3. DOM 和 CSSOM 合并成渲染

![](http://qiniucdn.luckybird.me/blog/img/2019/dom-cssom.png)


#### 4. 布局 Layout

##### 确定各个元素的位置，包括元素在视图中的坐标以及自身的大小，将其安置在浏览器的正确位置

- 计算此渲染对象的宽度 width；
- 遍历此渲染对象的所有子级，步骤跟本对象一样；
- 设置此渲染对象的高度 height，根据子渲染对象的累积高，margin 和 padding 的高度设置其高度；
- 设置此渲染对象脏位值为 false。


#### 5. 绘制 Painting
##### 将页面绘制到浏览器，设置各个元素的颜色，边框轮廓等，绘制顺序如下

- 背景颜色
- 背景图片
- 边框
- 轮廓



## 二，页面阻塞情况
---


##### 对于直接写在 html 页面的 CSS，会立即解析，但不阻塞 html 解析
##### 对于直接写在 html 页面的 JS，会立即解析和执行，会阻塞 html 和 css 解析
##### 对于外链 CSS 和 JS，分情况来分析

#### 1. 外链 CSS 下载和解析
- 在解析 html 时，如果遇到 CSS 外链，会立即发起下载请求，但不会阻塞 html 继续解析。
可能原因：页面还没到渲染步骤，为提高性能，可同时进行 html 解析和 CSS 下载。
- html 解析和 CSS 解析可以同时进行，不会相互阻塞。
可能原因：页面还没到渲染步骤，为提高性能，可同时进行 html 解析和 CSS 解析。
- 下载和解析 CSS，会阻塞 DOM 和 CSSOM 的合并渲染。
可能原因：渲染会用到 CSS 样式，所以必须先下载完 CSS，并完成 CSS 的解析，才开始渲染。
- 会阻塞后面 js 语句的执行。
可能原因：JS 也会修改 CSS，那么得先加载和解析前面的 CSS，然后在此基础上进行修改。


#### 2. 外链 JS 下载和执行
- 在解析 html 时，如果遇到 JS 外链，没有设置 defer 和 async 属性，那么会立即下载和执行，并阻塞 html 和 css 解析和渲染。
- 在解析 html 时，如果遇到 JS 外链，如果设置了 defer 属性，那么会异步下载，但不会立即执行，也不会阻塞 html 解析；等到 html 完成解析后，在触发 DOMContentLoaded 前，才按照加载顺序执行。 
- 在解析 html 时，如果遇到 JS 外链，如果设置了 async 属性，那么会异步下载，此时不会阻塞 html 解析，但是下载完后，会立即执行 JS 内容，不会按照加载的顺序，加载快的 JS 文件会更快执行，并且会阻塞 html 解析。


![](http://qiniucdn.luckybird.me/blog/img/2019/js-download.jpg)


## 三，页面回流和重绘
---


##### 当页面呈现完毕后，JS 可能会更改页面，此时页面会重新进行渲染，布局和绘制。


#### 1. 重排 reflow：当元素坐标位置和几何大小发生改变，或者 dom 发生变化时，需要重新渲染，布局，绘制页面，这个过程称为重排。


- 增加、删除、修改DOM结点时，会导致Reflow或Repaint
- 移动DOM的位置，或是搞个动画的时候。
- 修改CSS样式的时候。
- Resize窗口的时候，或是滚动的时候。
- 修改网页的默认字体时。
- display:none会触发reflow，而visibility:hidden只会触发repaint，因为没有发现位置变化。


#### 2. 重绘 repaint：当元素颜色，或者边框改变时，需要重新绘制页面，这个过程称为重绘，重绘成本远低于重排。


## 四，前端页面优化方法
---


#### 1. 资源加载


- 使用 dns 预解析，缓存到浏览器系统中，在 meta 添加 <link rel="dns-prefecth" href="https://www.google.com">
- 减少重定向，http://www.example.com 和 http://www.example.com/ 一样能够访问
- 使用缓存，如果资源没有更新，强制使用缓存，避免重复加载。
- 使用 cdn 部署静态资源，就近下载，减少物理距离产生的耗时。
- 通过合并 js，css，图片等，减少 http 请求。
- 通过后端压缩，减少 js，css，图片等文件大小。


#### 2. 页面呈现


- 为了避免阻塞 dom 和 cssom 解析，将 js 文件放到文档最后 </body> 加载，或者异步加载
- 页面渲染必须先解析完 cssom，所以把 css 放到文档最前面 <head> 加载。
- html 编写严格按照标准，必须包含开始和闭合标签，减少 dom 解析时间。
- css 选择器尽量简单，避免多层嵌套产生的深度遍历耗时。
- 减少 js 对 dom 操作，避免重复渲染和绘制页面。
