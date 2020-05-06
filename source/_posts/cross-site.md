---
title: JS 跨域请求
date: 2019-11-07 07:20:59
tags: JavaScript
---

>JS 向后端发起请求 HTTP 请求时，浏览器出于安全策略，禁止不同域名和端口之间的访问
>某些应用服务部署在不同域名下，网站需要访问不同域名下的服务，这是就产生的跨域问题

<!-- more -->


## 一，跨域由来
1. JS 向后端发起请求 HTTP 请求时，浏览器出于安全策略，禁止不同域名和端口之间的访问
2. 某些应用服务部署在不同域名下，网站需要访问不同域名下的服务，这是就产生的跨域问题


## 二，解决方案
### 1. 借助 img 标签的 src 属性
```html
<img src="http://www.otherdomain.com/myaction">
```
### 原理：
当 img 尝试加载 src 内容时，会发起 get 方式请求；由于 img 不对来源即域名进行强校验，所以能执行不同域名的 get 请求

### 缺点：
img 的加载方式只能是 get，而且无法获得 HTTP 请求的返回内容


## 2. 借助 script 加载文件的方式
```html
<!-- 前端加载 JS 文件 -->
<script type="text/javascript" src="http://www.otherdomain.com/callbackfunction.js"></script>
```
```php
// 后端返回字符串 alter('I am callbackfunction')
echo "alter('I am callbackfunction')";

```

```js
// 前端直接把字符串当做 JS 文件内容执行
alter('I am callbackfunction');

```

### 原理：
因为 script 加载 src 文件也是不强校验域名，所以能访问其他域名的服务；src 返回的是一段可执行的 JS 代码，比如一个 callback 函数

### 升级：
callback 函数可以自定义，传递给后端；后端给函数追加参数后，再传给前端执行；jQuery 已经把该方式添加到 Ajax 函数，并叫做 JSONP，但其实跟 Ajax 原理是不一样的，Ajax 底层是通过 XMLHttpRequest

```js

// JQuery 把跨域请求包装到 ajax，用数据类型 JSONP 区分，底层也是利用 script 加载 src 方式实现
$.ajax({
       type: "get",
       async: false,
       url: "http://www.otherdomain.com/myaction",
       dataType: "jsonp",
       jsonp: "callback",
       jsonpCallback:"?",
       success: function(result){
         alert(result.name);
       },
       error: function(){
       }
     });

// 另一种包装方式
$.getJSON("http://www.otherdomain.com/myaction?jsonpcallback=?", function(result) {
    alert(result.name);
});
```

### 缺点：
script 也是 get 方式加载内容，不支持 post 方式


## 3. 通过 CORS 方式

### 原理
浏览器出于安全考虑，访问跨域资源，获得内容后，会校验 HTTP 的头部信息，如果不合符规范，则不展示，即拦击掉。
Cross-Origin Resource Sharing，跨域资源共享，允许浏览器根据服务器设置的 HTTP 头部访问规则，进行内容展示还是拦截。
由此可知 CORS 需要资源服务器进行 HTTP 的访问设置，是否允许不同域的不同方法访问。

服务器 nginx 的 CORS 访问设置
```sh
# HTTP 来源
Access-Control-Allow-Origin:*
# 访问方法
Access-Control-Allow-Methods:*
# 更多设置 ···
```

### 优点
支持 post , get , put 等多种方式的 http 请求

### 缺点
需要资源服务器进行 http 访问配置


## 4. 通过 WebSocket 方式

### 原理
WebSocket 协议支持不同域访问

### 缺点
后端需要搭建一套 socket 服务，前端要维护 socket 状态
