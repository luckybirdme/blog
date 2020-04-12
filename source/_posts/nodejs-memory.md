---
title: nodejs 内存泄露
date: 2020-01-29 18:22:06
tags: nodejs
---

> nodejs 内存泄露

<!-- more -->

## 一. 内存使用和回收机制

### 1. 内存使用
nodejs 可通过 process.memoryUsage() 查看内存使用情况，其中 heapUsed 就是 V8 堆内存使用情况，即 JS 变量等使用的内存

```js
{
  rss: 19353600,
  heapTotal: 4734976,
  heapUsed: 2437808,
  external: 844606
}
```

- rss（resident set size）：所有内存占用，包括指令区和堆栈。
- heapTotal："堆"占用的内存，包括用到的和没用到的。
- heapUsed：用到的堆的部分。
- external： V8 引擎内部的 C++ 对象占用的内存。


### 2. 回收机制
- 对于 C 语言，需要手动申请和释放内存，但是 JS 有 "垃圾回收机制"（garbage collector），简称 gc
- JS 最常用的回收机制是"引用计数"（reference counting），如果变量不再引用（使用）了，则对其内存进行回收。


## 二. 内存泄露的案例和解决方法

### 1. 全局变量
全局变量，如果没有设置为 null 或者 undefined, 不会被回收
```js
// 定义全局变量
global var1 = 'var1'
global.var2 = 'var2'
global.var3 = []

// 释放全局变量
global.var1 = null
global.var2 = undefined
delete global.var3
```

### 2. 缓存对象
缓存对象如果没有清理机制，会不停地增加，导致内存泄露
```js

function format (bytes) {
  return (bytes / 1024 / 1024).toFixed(2) + ' MB'
}

// 全局缓存变量，如果没有设置为 null 或者 undefined, 不会被回收
let global_cache = []
let global_func = function () {
  // 根据时间，缓存对象不停地增加
  global_cache[new Date().getTime()] = new Array(1000000000).join('*')
  
  // 定期清理缓存, 避免缓存对象过大
  // global_cache = [];

  // 手动触发GC，保证能回收的内存都回收了
  global.gc() 
  console.log(`heapUsed: ${format(process.memoryUsage().heapUsed)}`)
}

// 显示内存使用情况
setInterval(global_func, 100)
```


### 3. 闭包引用
经典示例[https://github.com/ElemeFE/node-interview/issues/7](https://github.com/ElemeFE/node-interview/issues/7)

- 内存泄露示例代码

```js
function format (bytes) {
  return (bytes / 1024 / 1024).toFixed(2) + ' MB'
}

let theThing = null
let replaceThing = function () {
  let leak = theThing
  // 内存泄露代码
  let unused = function () {
    if (leak) { console.log('hi') }
  }
  // 不断修改引用
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log('a')
    }
  }

  global.gc() // 手动触发GC，保证能回收的内存都回收了
  console.log(`heapUsed: ${format(process.memoryUsage().heapUsed)}`)
}
setInterval(replaceThing, 100)

```

- 解决方法

```js
// 解决方法一：删除leak变量，直接引用最外层的theThing
let theThing = null
let replaceThing = function () {
  let unused = function () { // eslint-disable-line
    if (theThing) { console.log('hi') }
  }
  // 不断修改引用
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log('a')
    }
  }

  global.gc() // 手动触发GC
  console.log(`heapUsed: ${format(process.memoryUsage().heapUsed)}`)
}
setInterval(replaceThing, 100)


// 解决方法二：将unused方法引用的变量改为参数传递
let theThing = null
let replaceThing = function () {
  let leak = theThing
  let unused = function (thing) { // eslint-disable-line
    if (thing) { console.log('hi') }
  }
  // 不断修改引用
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log('a')
    }
  }

  global.gc() // 手动触发GC
  console.log(`heapUsed: ${format(process.memoryUsage().heapUsed)}`)
}
setInterval(replaceThing, 100)
```

### 4. 监听函数
在 nodejs 中，事件监听使用 event 类，它包含一个 listeners 数组，存放所有的监听回调的方法，listeners 组件默认最大值为 10, 避免 listener 过多产生内存泄露

- listener 过多示例代码

```js
const net = require('net')
let client = new net.Socket()

function format (bytes) {
  return (bytes / 1024 / 1024).toFixed(2) + ' MB'
}

function callbackListener () {
  console.log('connected!')
};

function connect () {
  client.connect(8000, '127.0.0.1', callbackListener)
}

connect()

client.on('error', (error) => { // eslint-disable-line
  console.log(`heapUsed: ${format(process.memoryUsage().heapUsed)}`)
})

client.on('close', function () {
  client.destroy()
  setTimeout(connect, 1)
})

```

- 解决方法

```js
client.on('close', function () {
  // 移除之前的监听器
  client.removeListener('connect', callbackListener);
  client.destroy()
  setTimeout(connect, 1)
})

```


### 内存分析工具可参考 Easy-Monitor 