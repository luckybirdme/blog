---
title: js http lib
date: 2019-12-07 20:34:09
tags: JavaScript
---

> XMLHttpRequest, 原生 JS 方法实现，最原始的方式。
> jQuery ajax, 对 XMLHttpRequest 进行包装，更方便使用。
> axios, 采用 Promise 链式对 XMLHttpRequest 进行包装，符合 ES6 规范。
> Fetch API, 是对 HTTP 接口的抽象接口，类似 XMLHttpRequest, 但是更底层，更具可扩展性和高效性。

<!-- more -->


## 一. XMLHttpRequest

#### 现代浏览器最开始与服务器交换数据，都是通过 XMLHttpRequest 对象。它可以使用 JSON,XML,HTML 和 text 文本等格式发送和接收数据。

#### 1. 优点
- 不重新加载页面的情况下更新网页
- 在页面已加载后从服务器请求/接收数据
- 在后台向服务器发送数据。

#### 2. 缺点：
- 使用起来也比较繁琐，需要设置很多值。
- 早期的IE浏览器有自己的实现，这样需要写兼容代码。

```js
if (window.XMLHttpRequest) { 
  // model browser
  xhr = new XMLHttpRequest()
} else if (window.ActiveXObject) { 
  // IE 6 and older
  xhr = new ActiveXObject('Microsoft.XMLHTTP')
}
xhr.open('POST', url, true)
xhr.send(data)
xhr.onreadystatechange = function () {
  try {
    // TODO 处理响应
    if (xhr.readyState === 4) {
      // Everything is good, the response was received.
      if (xhr.status === 200) {
        // Perfect!
      } else {
        // There was a problem with the request.
        // For example, the response may hava a 404 (Not Found)
        // or 500 (Internal Server Error) response code.
      }
    } else {
      // Not ready yet
    }
  } catch (e) {
    // 通信错误的事件中（例如服务器宕机）
    alert('Caught Exception: ' + e.description)
  }
}
```

## 二.  jQuery ajax
#### jQuery 将 XMLHTTPRequest 包装成 ajax, 兼容了各浏览器，可以有简单易用的方法 $.get，$.post。
#### 1. 优点：
- 对原生XHR的封装，做了兼容处理，简化了使用。
- 增加了对 JSONP 的支持，可以简单处理部分跨域。

#### 2. 缺点：
- 如果有多个请求，并且有依赖关系的话，容易形成回调地狱。
- ajax是jQuery中的一个方法。如果只是要使用ajax却要引入整个jQuery非常的不合理。

```js
$.ajax({
  type: 'POST',
  url: url,
  data: data,
  dataType: dataType,
  success: function () {},
  error: function () {}
})
```


## 三. [axios](https://github.com/axios/axios)
#### axios是一个基于promise的HTTP库，可以用在浏览器和 node.js 中。它本质也是对原生XMLHttpRequest的封装，只不过它是Promise的实现版本，符合最新的ES规范。

#### 1. 优点：
- 从浏览器中创建XMLHttpRequests, 从 node.js 创建 http 请求
- 支持 Promise API, 链式调用
- 拦截请求和响应，并支持取消请求
- 转换请求和返回数据，比如自动转换 json 格式
- 客户端支持防御 XSRF
- 提供了一些并发请求的接口

#### 2. 缺点：
- axios 基于 promise, 而 promise 是 ES6 规范，所以只持现代代浏览器。

```js
axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .finally(function () {
    // always executed
  }); 
```


## 四. [Fetch API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API)
#### Fetch API提供了一个 JavaScript 接口，用于访问和操作HTTP管道的部分，例如请求和响应。它还提供了一个全局fetch()方法，该方法提供了一种简单，合理的方式来跨网络异步获取资源。
#### fetch 是低层次的API，代替XHR，可以轻松处理各种格式，非文本化格式。可以很容易的被其他技术使用，例如Service Workers。但是想要很好的使用fetch，需要做一些封装处理。

#### MDN 强调 Fetch 和 jQuery ajax 的区别
- 当接收到一个代表错误的 HTTP 状态码时，从 fetch()返回的 Promise 不会被标记为 reject， 即使该 HTTP 响应的状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve （但是会将 resolve 的返回值的 ok 属性设置为 false ），仅当网络故障时或请求被阻止时，才会标记为 reject。
- 默认情况下，fetch 不会从服务端发送或接收任何 cookies, 如果站点依赖于用户 session，则会导致未经认证的请求（要发送 cookies，必须设置 credentials 选项）。

#### 1. 优点
- 符合关注分离，没有将输入、输出和用事件来跟踪的状态混杂在一个对象里
- 更加底层，提供的API丰富（request, response）
- 脱离了XHR，是ES规范里新的实现方式


#### 2. 缺点
- fetchtch只对网络请求报错，对404，500都当做成功的请求，需要封装去处理
- fetch默认不会带cookie，需要添加配置项
- fetch不支持abort，不支持超时控制，使用setTimeout及Promise.reject的实现的超时控制并不能阻止请求过程继续在后台运行，造成了量的浪费
- fetch没有办法原生监测请求的进度，而XHR可以
- 目前浏览器兼容性不大好。

```js

fetch('/users.json', {
    method: 'post', 
    data: {}
}).then(function() { /* handle response */ });
```