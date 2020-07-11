---
title: websocket 和 http 区别
date: 2020-06-29 07:53:40
tags: Linux
---

> websocket

<!-- more -->


## 一. http 交互过程

1. 浏览器向服务端发起http请求，并携带 request 的 header 
2. 底层采用tcp协议，三次握手后，服务端将数据返回给浏览器，并携带 response 的 header
3. 浏览器收到服务端数据，如果是短连接，则直接断开tcp连接，如果是长连接，则保持，以便其他http请求共用
4. 每次交互必须先由浏览器主动发起，服务端无法主动发送信息给浏览器。


![](/img/2020/websocket_1.jpg)


## 二. websocket 交互过程
1. 浏览器向服务端发起http请求，并携带 request 的 header，要求建立 websocket 连接。
2. 底层采用tcp协议，三次握手后，服务端同意建立 websocket 链接，后续采用 socket 协议。
3. 当 websocket 链接建立后，浏览器可以向服务端发送数据，服务端也可以向浏览器发送数据。
4. 数据通过帧传递，减少不必要 header，因此效率更高。

![](/img/2020/websocket_2.jpg)


## 三. http和websocket对比

分类|http|websocket
-|-|-
通道|单向，只有浏览器可以主动发送信息给服务端|全双通信，两端都可以主动发送信息
连接|一次请求后，短连接会马上断开TCP连接|可以保持一直连接，直到某一方主动断开为止
信息|包括较大的 header | header 更小，而且采用帧方式传递






