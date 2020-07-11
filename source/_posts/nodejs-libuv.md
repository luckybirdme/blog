---
title: JS Event Loop 任务执行机制
date: 2020-01-30 14:26:20
tags: JavaScript
---

> Event Loop

<!-- more -->


## 一. Event Loop 执行的任务简介

1. 任务队列又分为macro-task（宏任务）与micro-task（微任务），在最新标准中，它们被分别称为task与jobs。

2. macro-task大概包括：script(整体代码), setTimeout, setInterval, setImmediate, I/O, UI rendering。

3. micro-task大概包括: process.nextTick, Promise, MutationObserver(html5新特性)


执行过程：
执行一个宏任务(先执行同步代码)-->执行所有微任务-->UI render-->执行下一个宏任务-->执行所有微任务-->UI render-->......

![](/img/2020/event_loop_1.png)




## 二. 具体事件函数分析

1. process.nextTick 
一般用于同步方法执行完后，在队列执行前，做一些操作


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



2. setImmediate 
一般用于异步队列任务执行完毕后，做一些操作。


- 与 nextTick 执行顺序的区别

```js

A();
B();
C();

```

![](/img/2020/nodejs_event_run_one.jpg)

```js
A();
process.nextTick(B);
C()

```

![](/img/2020/nodejs_event_run_two.jpg)


```js
A();
setImmediate(B);
C();
```

![](/img/2020/nodejs_event_run_three.jpg)



- 与 setTimeout 执行顺序比较复杂，可能在前也可能在后

```js

const fs = require( "fs" );

// 同步方法读文件
let data = fs.readFileSync( "hello.txt", "utf8" ).toString()
console.log('readFileSync' ,data);
console.log( "同步读取完毕" );

// 将会在同步方法执行完后，马上执行
process.nextTick(function(){
  console.log('nextTick')
} );

// 异步读取文件，将会在 nextTick 执行后才返回结果
fs.readFile( "hello.txt", "utf8", (err,data)=>{
  if(err) throw err;
  console.log('readFile' ,data);
  console.log( "异步读取完毕" );

  setTimeout(function(){
    console.log('setTimeout 2')
  },0)

  setImmediate(function(){
    console.log('setImmediate, 此时队列为空，比 setTimeout 优先执行')
  });

} )

setImmediate(function(){
  console.log('setImmediate, 此时队列有异步读取文件，比 setTimeout 晚执行')
});

setTimeout(function(){
  console.log('setTimeout 1')
},0)


```


使用场景：
- process.nextTick, 将会当前事件循环最后，进入下一个事件循环前优先执行，所以效率最高，但会阻塞后续任务调用。
- setTimeout, 需要读取计时器，等待时间跟线程处理性能相关，所以具有更多不确定性。而且因为动用了红黑树，所以消耗资源大。
- setImmediate, 是在事件循环结束后执行的，如果等待时间为零的 setTimeout 任务, 建议使用 setImmediate 替代。



