---
title: 异步任务队列
date: 2020-06-27 16:49:09
tags: JavaScript
---

> 异步任务队列

<!-- more -->


### 1. 利用 promise 实现异步任务队列，并发地执行异步任务，且等待所有返回结果后一起处理，利用 Promise.all 函数来实现

```js
function myConsole(...args){
  console.log(new Date(),...args)
}

function f(v) {
  // 初始化 f('f1') 时就会执行
  myConsole('f',v)
  return new Promise((resolve, reject) => {
  	// 初始化 f('f1') 时就会执行
    myConsole('Promise',v)
    setTimeout(function() {
      myConsole('setTimeout',v)
      resolve(v)
    }, 1000)
  })
}

// f('f1') 的异步任务已经开始执行了
func=[f('f1'),f('f2'),f('f3')]

// 这里是并发地执行异步任务，且并发地处理结果
// 并发地执行，所有都成功才进入 then，否则进入 catch
Promise.all(func).then((v)=>{
  myConsole('then',v)
}).catch((e)=>{
  myConsole('catch',e)
})

```


### 2. 利用 promise 实现异步任务队列，并发地执行异步任务，但是串行地处理返回结果，利用 then 链式调用来实现

```js

// f('f1') 的异步任务已经开始执行了
func=[f('f1'),f('f2'),f('f3')]

// 并发地初始化 f('f1')，且并发地执行异步任务，但是处理结果是串行
func[0].then((v) => {
  myConsole(v)
  return func[1]
}).then((v) => {
  myConsole(v)
  return func[2]
}).then((v) => {
  myConsole(v)
})


```


### 3. 利用 promise 实现串行地执行任务，且串行地处理返回结果，也是利用 then 链式调用来实现，只是每次都等待上一次 promise 执行完才初始化下一个 promise

```js

// 串行地初始 f('f1')，串行地执行异步任务，且串行地处理结果
f('f1').then((v) => {
  myConsole(v)
  return f('f2')
}).then((v) => {
  myConsole(v)
  return f('f3')
}).then((v) => {
  myConsole(v)
})

```

### 4. 如果不确定有多个异步任务，或者是动态的异步任务数，就无法手写 then 实现串行执行了，需要一个自动递归调用的方法

```js
// 自动递归调用 promise 
function sequence(promises, cb, ...args) {
    const p = Promise.resolve(),
        len = promises.length
    if (len <= 0) {
        return p
    }
    let i = 0
    //如果cb不是函数
    if (typeof cb !== 'function') {
        cb = null
        args = [cb, ...args]
    }

    function callBack(...params) {
        return p.then(r => {
            return promises[i](r, ...params)
        }).then(r => {
            ++i
            cb && cb(r, i, ...params)
            return i > len - 1 ? Promise.resolve(r) : callBack(...params)
        })
    }

    return callBack(...args)
}


var list = [];
for (let i = 0; i < 5; ++i) {
    list.push(i);
}

var promises = list.map( number=>{
    // 这里初始化会执行
    myConsole('list',number)
    // 返回闭包函数，所以不会立即执行
    return  _ => new Promise((resolve, reject) => {
            setTimeout(() => {
                myConsole('Promise',number)
                return resolve(number);
            }, 1000);
        })
});
sequence(promises).then((data) => {
    myConsole('then',data)
}).catch((err) => {
    console.log(err)
});

```


### 5. 上述编写逻辑比较复杂，其实 ES7 已经在编程语言上支持异步排队调用了

```js
var list = [];
for (let i = 0; i < 5; ++i) {
    list.push(i);
}

var promises = list.map( number=>{
    // 这里初始化会执行
    myConsole('list',number)
    // 返回闭包函数，所以不会立即执行
    return  _ => new Promise((resolve, reject) => {
            setTimeout(() => {
                myConsole('Promise',number)
                return resolve(number);
            }, 1000);
        })
});
// 立即调用
// 采用匿名函数
// async/await 异步编程关键字
(async () => {
    for(let i=0;i<promises.length;i++) {
        // 执行到这里时，会等待 promise 执行完毕
        // 等待过程会切换到出去，所以并不会阻塞主线程
        // 相当于执行 promises 里的闭包函数
        await promises[i]();
    }
    console.log('all well done');
})();

```

### 6. 如果想并发地执行，但是串行地处理结果，则 promises 不要闭包函数

```js

var list = [];
for (let i = 0; i < 5; ++i) {
    list.push(i);
}

var promises = list.map( number=>{
    // 这里初始化会执行
    myConsole('list',number)
    // 并发地执行，直接执行
    return new Promise((resolve, reject) => {
    	// 初始化 promise 时会立即执行
        myConsole('Promise',number);
        setTimeout(() => {
            myConsole('setTimeout',number)
            return resolve(number);
        }, (4-number)*1000);
    })
});

// 立即调用
// 采用匿名函数
// async/await 异步编程关键字
(async () => {
    for(let i=0;i<promises.length;i++) {
        // 执行到这里时，会等待 promise 执行完毕
        // 等待过程会切换到出去，所以并不会阻塞主线程
        rtn = await promises[i];
        myConsole('await done i : ',i,rtn);
    }
    console.log('all well done');
})()

```
