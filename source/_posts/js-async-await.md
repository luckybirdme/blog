---
title: js 异步编程
date: 2020-01-18 20:23:15
tags: JavaScript
---

> js async/await 属于 ES7 标准，目的是为了编写异步代码，跟同步代码一样。
> await  操作符用于等待 Promise 对象状态变化，只能在异步函数 async function 中使用。

<!-- more -->


### 1. 按照顺序加载图片，多层回调地狱

```js
// 图片地址
var imgUrl=[
    '../img/0.jpg',
    '../img/1.jpg',
    '../img/2.jpg',
    '../img/3.jpg'
    ];

// 加载图片
var loadImg = function(name,url,callback){
    var img = new Image();
    img.src = url;
    img.onload = function() {
        console.log(name + ' success');
        callback();
    }
    img.onerror = function() {
        console.log(name + ' error');
    }
}

// 按照顺序，异步加载图片
loadImg('img0',imgUrl[0],function(){
    loadImg('img1',imgUrl[1],function(){
        loadImg('img2',imgUrl[2],function(){
            loadImg('img3',imgUrl[3],function(){
                console.log('load all img');
            })
        })
    })
})
```

### 2. 通过 promise 链式调用，但依然有回调的复杂过程。

``` js

// 通过 promise 实现图片加载
var imgPm = function(name,url){
    var img = new Image();
    img.src = url;
    return new Promise(function(resolve,reject){
        img.onload = function() {
            resolve(name + ' success');
        }
        img.onerror = function() {
            reject(name +' error');
        }
    });
    
};

// 通过 then 链式回调    
imgPm('img0',imgUrl[0])
.then(function(msg){
    return imgPm('img1',imgUrl[1]);
})
.then(function(msg){
    return imgPm('img2',imgUrl[2]);
})
.then(function(msg){
    return imgPm('img3',imgUrl[3]);
})
.then(function(msg){
    console.log('all done');
});
```

### 3. 通过生成器和 yield 关键字模拟异步调用


```js

// 通过生成器和 yield 进行异步加载图片，代码编写好像同步代码一样
function* imgGen() {
    yield imgPm('img0',imgUrl[0]);
    yield imgPm('img1',imgUrl[1]);
    yield imgPm('img2',imgUrl[2]);
    yield imgPm('img3',imgUrl[3]);
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

// 执行生成器，加载图片
run(imgGen);
```


### 4. 生成器的执行函数 run, 可使用现场的类库 [co](https://github.com/tj/co), co 是 co是coroutine的简称，借鉴于python、lua等语言中的协程概念，即生成器和 yield 模拟实现了协程切换，它可以将异步的代码逻辑写成同步的方式，这使得代码的阅读和组织变得更加清晰，也便于调试。 

``` js
// npm install --save co
import co from 'co';
co(imgGen);

```

### 5. 虽然 co 是社区里面的优秀异步解决方案，但并不是语言标准，只是一个过渡方案。ES7 语言层面提供 async / await 关键字去解决语言层面的难题。

```js

// 异步调用函数，将会通过事件循环异步执行，但是编写语法跟同步代码非常类似
async function imgAsync() {
  await imgPm('img0',imgUrl[0]);
  await imgPm('img1',imgUrl[1]);
  await imgPm('img2',imgUrl[2]);
  await imgPm('img3',imgUrl[3]);
}

// 直接调用异步函数，系统自动识别，并通过事件循环来异步执行
imgAsync()
.then(x => console.log(x))
.catch(err => console.error(err));
```