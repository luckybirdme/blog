---
title: JS web worker
date: 2020-06-27 08:36:27
tags: JavaScript
---

> JS web worker

<!-- more -->


## 一. JS 单线程运行机制
JavaScript 语言采用的是单线程模型，一次只能做一件事，前面的任务没做完，后面的任务只能等着。比如，当进行大量数据计算时，JavaScript会卡住整个页面，导致无法进行UI交互。

为了解决单线程的缺陷，浏览器就有了 Web Worker 的多线程使用，它允许在JavaScript主线程外，创建多个独立的线程，执行耗时的JS任务，避免阻塞主线程。


## 二. Web Worker 的特点

1. 同源限制
分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源，即无法进行跨域的JS脚本执行。

2. DOM 限制
Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用document、window、parent这些对象。但是，Worker 线程可以navigator对象和location对象。

3. 通信联系
Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成，传递参数也不能引用传递，必须拷贝一份副本，所以在大文件拷贝时需要特别注意性能，可以考虑使用 ShareArrayBuffer 跟主线程共享数据。

4. 脚本限制
Worker 线程不能执行alert()方法和confirm()方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。

5. 文件限制
Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。


## 三. Web Worker 的使用

1. 主线程，创建 Worker 线程

```js
// work.js 必须同源路径下
var worker = new Worker('work.js');
```

2. 主线程，发送消息给 Worker 线程

```js
worker.postMessage('Hello World');
worker.postMessage({method: 'echo', args: ['Work']});
```


3. worker 线程，侦听主线程的消息

```js

onmessage = function(e) {
  console.log('Message received from main script');
  var workerResult = 'Result: ' + (e.data);
  console.log('Posting message back to main script');
  postMessage(workerResult);
}
```

4. worker 线程，发送消息给主线程

```js
postMessage('You said: ' + e.data);
```

5. 主线程，接收 worker 线程的信息

```js
worker.onmessage = function(e) {
  console.log('Message received from main script');
  var workerResult = 'Result: ' + (e.data);
  console.log('Posting message back to main script');
  postMessage(workerResult);
}

```


6. worker 线程，可加载额外的JS脚本

```js

importScripts('script1.js');
importScripts('script1.js', 'script2.js');

```

7. 主线程，监听 worker 线程的异常

```js

worker.onerror(function (e) {
  console.log(e);
});
```

8. 最后记得删除 worker 线程

```js
// 主线程
worker.terminate();

// Worker 线程
self.close();
```


## 四. worker 线程使用场景

1. 大量计算，比如加密算法，图片base64转码
2. 预取数据，因为不影响JS主线程的UI交互，所以可以后台预先下载数据，以备后续使用
3. 高频交互操作，比如用户输入后，进行复杂且多次的校验，为了避免UI交互卡顿，可以通过 worker 来校验，用户无感知