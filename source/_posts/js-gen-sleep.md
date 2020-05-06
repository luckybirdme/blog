---
title: js 生成器实现定时器
date: 2020-01-18 20:23:15
tags: JavaScript
---

> 生成器实现定时器 

<!-- more -->

### 1. 通过生成器实现定时器
```js
// 返回 promise
function sleep(time) {
  return new Promise(function(resolve,reject){
    setTimeout(resolve,time);
   })
} 
// 生成器
function* gen() {
  console.log(1);
  // 睡眠
  yield sleep(1000)
  console.log(2);
  // 睡眠
  yield sleep(2000)
  console.log(3);
}

let it = gen()
it.next().value.then((resolve,reject)=>{
  console.log('sleep end 1')
  it.next().value.then((resolve,reject)=>{
    console.log('sleep end 2')
  })
})

```

### 2. 上述例子需要手动执行 next，改造成自动执行

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