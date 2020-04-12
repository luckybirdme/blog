---
title: js yield 
date: 2020-01-18 20:23:15
tags: JavaScript
---

> js yield 关键字搭配 generator function 一起使用

<!-- more -->

### 1. [generator function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator) 和 yield (https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield) 搭配使用，创建迭代器。
```js
var generator = function* () {
    yield 'a';
    yield 'b';
    yield 'c';
};
var it = generator()
console.log(it.next()); 
console.log(it.next()); 
console.log(it.next());
console.log(it.next());
/*
{ value: 'a', done: false }
{ value: 'b', done: false }
{ value: 'c', done: false }
{ value: undefined, done: true }
*/
```

### 2. 通过迭代器实现定时器
```js
// 返回 promise
function sleep(time) {
  return new Promise(function(resolve,reject){
    setTimeout(resolve,time);
   })
} 
// 迭代器
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

### 3. 上述例子需要手动执行 next，改造成自动执行

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