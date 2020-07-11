---
title: JS 内存泄露
date: 2020-01-29 18:22:06
tags: JavaScript
---

> JS 内存泄露

<!-- more -->

## 一. JS 内存生命周期

JavaScript在创建变量（对象，字符串，函数等）时分配内存，并且在不再使用变量时“自动”释放内存，这个自动释放内存的过程称为垃圾回收。

生命周期如下：
- 内存分配：当我们申明变量、函数、对象的时候，系统会自动为他们分配内存
- 内存使用：即读写内存，也就是使用变量、函数等
- 内存回收：使用完毕，由垃圾回收机制自动回收不再使用的内存

![](/img/2020/js_memory.png)

## 二. JS 内存回收机制

### 1. 引用计数垃圾收集
这是最初级的垃圾收集算法。此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。

官网实例

```js

var o = { 
  a: {
    b:2
  }
}; 
// 两个对象被创建，一个作为另一个的属性被引用，另一个被分配给变量o
// 很显然，没有一个可以被垃圾收集


var o2 = o; // o2变量是第二个对“这个对象”的引用

o = 1;      // 现在，“这个对象”只有一个o2变量的引用了，“这个对象”的原始引用o已经没有

var oa = o2.a; // 引用“这个对象”的a属性
               // 现在，“这个对象”有两个引用了，一个是o2，一个是oa

o2 = "yo"; // 虽然最初的对象现在已经是零引用了，可以被垃圾回收了
           // 但是它的属性a的对象还在被oa引用，所以还不能回收

oa = null; // a属性的那个对象现在也是零引用了
           // 它可以被垃圾回收了
```


引用计算的机制存在循环引用无法回收的缺点，官网案例如下

```js
function f(){
  var o = {};
  var o2 = {};
  o.a = o2; // o 引用 o2
  o2.a = o; // o2 引用 o

  return "azerty";
}

f();
```

### 2. 标记清除算法
这个算法假定设置一个叫做根（root）的对象（在Javascript里，根是全局对象）。垃圾回收器将定期从根开始，找所有从根开始引用的对象，然后找这些对象引用的对象……从根开始，垃圾回收器将找到所有可以获得的对象和收集所有不能获得的对象。



## 三. 查看 JS 使用的内存情况

### 1. nodejs
nodejs 可通过 process.memoryUsage() 查看内存使用情况，其中 heapUsed 就内存使用情况，即 JS 变量等使用的内存

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


### 2. 浏览器
Chrome 浏览器通过 performance 的工具，可以查看

![](/img/2020/js_memory_1.png)



## 四. 内存泄露的案例和解决方法

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
