---
title: nodejs 事件轮询机制
date: 2020-01-30 14:26:20
tags: nodejs
---

> nodejs 事件驱动和非阻塞I/O通过 libuv 的实现
> libuv 包括轮询机制和异步I/O操作

<!-- more -->

JS 运行在 V8 引擎上，属于单线程，但是底层通过 libuv 实现多线程 IO 读写。
Windows 下使用 IOCP 来完成异步 IO，LINIX 上使用 libev 来实现。
多个 IO 任务会放到队列中，通过轮询执行多个线程读写，以及结果回调。

## 一. libuv 事件轮询机制主要分为六个阶段：

### 1. timers 计时器阶段
执行的是setTimeout()和setInterval()的回调

### 2. pending callbacks
某些系统操作（如tcp错误类型）的回调函数

### 3. idle ，prepare
nodejs 内部函数调用

### 4. poll 轮询阶段（轮询队列）
- 如果轮询队列不为空，依次同步取出轮询队列中第一个回调执行，知道轮询队列为空或者达到系统最大的限制
- 如果轮询队列为空
 - 如果之前设置过setImmediate函数，直接进入下一个check阶段
 - 如果之前没有设置过setImmediate函数，在当前poll阶段等待，直到轮询队列添加回调函数，就去第一个情况执行
 - 如果定时器setTimeout()和setInterval()到点了，也会去下一个阶段

### 5. check 查阶段
执行 setImmediate 设置的回调函数

### 6. close callbacks 关闭阶段
执行close时间回调函数


## 二. process.nextTick 可以在事件轮询任意阶段前执行，一般用于同步方法执行完后，在队列执行前，做一些操作


- 测试案例
```js
const fs = require( "fs" );

// 将会在同步方法执行完后，马上执行
process.nextTick(function(){
  console.log('nextTick')
} );

// 同步方法读文件
console.log( fs.readFileSync( "hello.txt", "utf8" ).toString() );
console.log( "同步读取完毕" );
// 异步读取文件，将会在 nextTick 执行后才返回结果
fs.readFile( "hello.txt", "utf8", (err,data)=>{
  if(err) throw err;
  console.log(data)
  console.log( "异步读取完毕" );
} )

```


- 具体的应用案例
```js
const server = net.createServer(() => {}).listen(8080);
server.on('listening', () => {});
```
当只有一个端口作为参数传入，端口会被立即绑定。所以监听回调可能被立即调用。问题是 on('listening') 回调在那时还没被注册。
为了解决这个问题，把listening事件加入到 nextTick() 队列中以允许脚本先执行完（同步代码）。这允许用户（在同步代码中）设置任何他们需要的事件处理函数。


## 三. setImmediate 在 poll 轮询阶段队列为空时，被拉起执行，一般用于异步队列任务执行完毕后，做一些操作。

- 与 setTimeout 执行顺序的区别的案例

```js

const fs = require( "fs" );
// 在 poll 轮询阶段队列为空时执行
setImmediate(function(){
  console.log('setImmediate, 此时队列有异步读取文件，比 setTimeout 晚执行')
});

// 在 timer 和 poll 阶段，到点时执行
setTimeout(function(){
  console.log('setTimeout')
},0)

// 同步方法读文件
console.log( fs.readFileSync( "hello.txt", "utf8" ).toString() );
console.log( "同步读取完毕" );
// 异步读取文件，将会在 nextTick 执行后才返回结果
fs.readFile( "hello.txt", "utf8", (err,data)=>{
  if(err) throw err;
  console.log(data)
  console.log( "异步读取完毕" );

  // 在 timer 和 poll 阶段，到点时执行
  setTimeout(function(){
    console.log('setTimeout')
  },0)

  // 在 poll 轮询阶段队列为空时执行
  setImmediate(function(){
    console.log('setImmediate, 此时队列为空，比 setTimeout 优先执行')
  });


} )
```

setTimeout 需要读取计时器，而且等待时间跟线程处理性能相关，所以具有更多不确定性。
setImmediate 放到内部 I/O 循环中，会比 setTimeout 更早执行，而且不受线程性能影响。 