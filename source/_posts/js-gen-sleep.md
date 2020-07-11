---
title: js 实现 sleep 函数
date: 2020-01-18 20:23:15
tags: JavaScript
---

> sleep 函数

<!-- more -->

### 1. JS 是单线程，直接通过循环来实现，但是会导致整个线程卡住，网页会卡顿，nodejs会卡死

```js
function sleep(delay) {
  var start = (new Date()).getTime();
  while ((new Date()).getTime() - start < delay) {
    continue;
  }
}

console.log(new Date())
sleep(1000);
console.log(new Date())
sleep(1000);
console.log(new Date())
sleep(1000);
console.log(new Date())

```


### 2. 通过 setTimeout 函数来实现，此时不会导致 JS 线程卡住，但是存在回调地狱的弊端。
备注：setTimeout 只在指定时间后执行一次，setInterval 以指定时间为周期循环执行，直到被 clearInterval 为止。

```js
function sleep(ms, callback) {
    setTimeout(callback, ms)
}

console.log(new Date())
sleep(1000, () => {
    console.log(new Date())
    sleep(1000, ()=>{
        console.log(new Date())
        sleep(1000, ()=>{
            console.log(new Date())
        })
    })
})
```


### 3. 通过 promise 链式调用，但是依然存在回调函数。

```js

const sleep = time => {
    return new Promise(resolve => setTimeout(resolve,time)
)} 
console.log(new Date())
sleep(1000).then(()=>{
    console.log(new Date())
    return sleep(1000)
}).then(()=>{
    console.log(new Date())
    return sleep(1000)
})

```


### 4. 通过生成器和 yield 关键字，实现异步执行，但编写类似同步代码一样，就是实现过程有点复杂，只是一个过渡方案。

```js
function sleep(time) {
    return new Promise(function(resolve,reject){
        setTimeout(resolve,time);
    }) 
}

// 生成器，每次执行到 yield 时，执行 sleep 函数，同时会切换出来，不阻塞线程
// 异步执行的代码，编写跟同步类似
function* gen(){
    console.log(new Date())
    yield sleep(1000);
    console.log(new Date())
    yield sleep(1000);
    console.log(new Date())
}

// 编写生成器执行函数，以便自动循环调用 next 方法
function run(gen) {
    let g = gen()
    function next(data) {
        // 加载下一个图片
        let result = g.next(data)
        if (result.done) {
            return result.value
        }
        // 还有图片需要加载，继续链式调用
        result.value.then((data) => next(data))
    }
    // 首次默认执行
    next()
}

// 执行生成器
run(gen)

```


### 5. 通过 ES7 的新特性 async/await 关键字，从编程语言上支持异步执行代码，像同步代码一样的编写。

```js

function sleep(time) {
    return new Promise(resolve =>
        setTimeout(resolve,time)
    )
}
// 异步执行函数，通过事件循环调用
// 当遇到 await 时，将会执行 sleep 函数，并切换出来，避免阻塞线程
// 当下次事件循环调用时，如果 sleep 函数完毕，则继续往下执行
async function main() {
    console.log(new Date())
    await sleep(1000)
    console.log(new Date())
    await sleep(1000)
    console.log(new Date())
}

main();

```