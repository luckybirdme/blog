---
title: js async/await 
date: 2020-01-18 20:23:15
tags: JavaScript
---

> js async/await 属于 ES7 标准，目的是为了更便于编写异步执行的方法
> await  操作符用于等待 Promise 对象状态变化，只能在异步函数 async function 中使用。

<!-- more -->


### 1. 通过生成器实现定时器

```js
// 编写Generator执行器
function run(gen) {
    let g = gen()
    function next(data) {
        let result = g.next(data)
        if (result.done) {
            return result.value
        }
        result.value.then((data) => next(data))
    }
    // 首次默认执行
    next()
}

function sleep(time) {
  return new Promise(function(resolve,reject){
    setTimeout(resolve,time);
   })
} 
function* gen() {
  console.log(1);
  yield sleep(1000)
  console.log(2);
  yield sleep(2000)
  console.log(3);
}

run(gen);
```

### 2. 通过 async/await 实现会更简便

```js

function sleep(time) {
  return new Promise(function(resolve,reject){
    setTimeout(resolve,time);
   })
} 
async function gen() {
  console.log(1);
  // 等待 sleep 即 promise 状态变化
  await sleep(1000)
  console.log(2);
  await sleep(2000)
  console.log(3);
}

gen();

```