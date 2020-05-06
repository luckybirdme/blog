---
title: nodejs events 核心类
date: 2020-01-29 13:32:04
tags: nodejs
---

> nodejs 运行的核心是事件驱动机制，通过核心类 events 来实现

<!-- more -->


### 大多数 nodejs 核心 API 构建于异步事件驱动的架构，其中某些类型的对象（又称触发器，Emitter）会触发命名事件来调用函数（又称监听器，Listener）。nodejs 所有能触发事件的对象都是 EventEmitter 类的实例，基于 nodejs 的 events 核心类。

- 简单的事件驱动机制

```js
const EventEmitter = require('events');

// 继承 event 的事件驱动特性
class MyEmitter extends EventEmitter {}

// 实例化
const myEmitter = new MyEmitter();
// listener
myEmitter.on('event', () => {
  console.log('触发事件');
});

// emitter
myEmitter.emit('event');

```

- fs 通过 Stream 读取文件，异步侦听数据流，Stream 是可读的、可写的、或者可读可写的，所有 Stream 都是 EventEmitter 的实例。

```js
let data = '';
let readerStream = fs.createReadStream('hello.txt',{ encoding : 'utf-8', highWaterMark : 1});
readerStream.on('data', function(chunk) {
  console.log(chunk);
  data += chunk;
});
readerStream.on('end',function(){
  console.log("read end");
  console.log(data);
});

```


- http 创建服务器，其中 request 是继承于 Stream, Stream 又继承于 Event，因此 request 也有事件驱动的机制。

```js
const http = require('http');
let data = '';
http.createServer(function (request, response) {
  request.on('data', function (chunk) {
    console.log('data',chunk)
    data += chunk;
  });
  request.on('end', function () {
    console.log('request end',data);
  });
}).listen(8080);
```