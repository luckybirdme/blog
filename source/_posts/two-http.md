---
title: RESTful API 发生两次 http 的原因
date: 2020-06-29 08:26:36
tags: Linux
---

> 复杂的请求
> 跨域请求

<!-- more -->

### 1. 产生了复杂请求。
复杂请求对应的就是简单请求。简单请求的定义是：
- 请求方法是GET、HEAD或者POST，并且当请求方法是POST时，Content-Type必须是application/x-www-form-urlencoded, multipart/form-data或着text/plain中的一个值。
- 请求中没有自定义HTTP头部。所谓的自定义头部，在实际的项目里，我们经常会遇到需要在header头部加上一些token或者其他的用户信息，用来做用户信息的校验。

### 2. 发生了跨域。

官方将头部带自定义信息的请求方式称为带预检（preflighted）的跨域请求。在实际调用接口之前，会首先发出一个options请求，检测服务端是否支持真实的请求进行跨域的请求。真实请求在options请求中，通过request-header将 Access-Control-Request-Headers与Access-Control-Request-Method发送给后台，另外浏览器会自行加上一个Origin请求地址。服务端在接收到预检请求后，根据资源权限配置，在response-header头部加入access-control-allow-headers（允许跨域请求的请求头）、access-control-allow-methods（允许跨域请求的请求方式）、access-control-allow-origin（允许跨域请求的域）。另外，服务端还可以通过Access-Control-Max-Age来设置一定时间内无须再进行预检请求，直接用之前的预检请求的协商结果即可。浏览器再根据服务端返回的信息，进行决定是否再进行真实的跨域请求。这个过程对于用户来说，也是透明的。

另外在HTTP响应头，凡是浏览器请求中携带了身份信息，而响应头中没有返回Access-Control-Allow-Credentials: true的，浏览器都会忽略此次响应。
总结：只要是带自定义header的跨域请求，在发送真实请求前都会先发送OPTIONS请求，浏览器根据OPTIONS请求返回的结果来决定是否继续发送真实的请求进行跨域资源访问。所以复杂请求肯定会两次请求服务端。


[原文地址](https://www.cnblogs.com/mamimi/p/10602722.html)

