---
title: JS Promise 使用
date: 2019-10-13 12:39:02
tags: JavaScript
---

> JavaScript 是单线程语言，为了加快执行效率，会采用异步回调的方式执行。
> 但是多层异步回调太深，不方便代码编写，也不利用阅读，也就是常说的回调地狱。
> Promise 提供一种链式调用，避免回调地狱的问题。

<!-- more -->

### 一，多层回调
##### 1. 按照顺序加载图片，会出现回调地狱：
```javascript

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
// 按照顺序加载图片
// 多层回调
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

##### 2. 避免回调地狱，<a id="Promise-example">使用 Promise 方式实现</a>：
```javascript
// 通过 promise 实现
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

### 二，Promise 介绍
##### 1. 简介
- Promise 对象保存异步操作的结果。
- Promise 异步操作具有三种状态，Pending、Resolved 和 Rejected。
- Promise 对象状态的改变可能存在两种情况，Pending 到 Resolved，或者 Pending 到 Rejected。
- Promise 对象的状态一旦确定，那么就无法改变，要么是Resolved，要么是 Rejected。

Promise 链式调用图。
![](http://qiniucdn.luckybird.me/blog/img/2019/promise.png)


##### 2. 构造函数 Promise
Promise 简单异步调用
```javascript
var isSuccess=true;
// 构造函数，创建 Promise 实例
var p = new Promise(function(resolve, reject){
	// 异步调用
	setTimeout(function(){
		if(isSuccess){
			// 调用成功，从 Pending 变成 Resolved
			resolve('success');
		}else{
			// 调用失败，从 Pending 变成 Rejected
			reject('failed')
		}
	},10);
});

// Resovled
p.then(function(msg){
	console.log(msg)
},
// Rejected
function(e){
	console.log(e)
})
```
- resolve 是 Promise 内部提供的回调函数，在 Resolved 状态时的回调函数，最终会调用实例 p 的 then 的第一个参数（即回调函数），而且会透传 参数。 
- reject 也是 Promise 内部提供的回调函数，在 Rejected 状态时的回调函数，最终会调用实例 p 的 then 的第二个参数（即回调函数），而且会透传参数；如果 then 没有第二参数，则会一直遍历整个链式，直到存在第二参数的 then，或者 catch 函数（后续会解释）。


##### 3. Promise.resolve() 和 Promise.reject() 自带函数
- Promise.resolve 函数参数如果是字符串，返回一个 Resolved 状态的 Promise，如果是 Promise 对象，则会直接返回这个状态的对象。
- Promise.reject 函数返回一个 Rejected 状态的 Promise，参数支持字符串或者 new Error 形式。

##### 4. Promise.prototype.then() 原型函数
多个 then 的链式调用，请参考 [使用 Promise 方式实现](#Promise-example)

then 第一个参数是 Resolved 状态下的回调函数，第二参数可选，是 Rejected 状态下的回调函数，这两个回调函数的参数是 Promise 构造函数实例化确定状态时透传过来的，即 resolve('success') 和 reject('error')。

then 默认返回是 Promise 实例，所以可以链式调用，返回的 Promise 状态根据以下规则来确定：
- 如果回调函数没有返回，则 then 返回 Promise.resolve()。
- 如果回调函数有返回，而且非 thenable 的对象，则 then 返回 Promise.resolve(rtn) 。
- 如果回调函数返回 thenable 的对象，比如 Promise 实例，则直接返回这个 Promise 对象。
- 如果回调函数执行抛出异常，则返回 Promise.reject(error)。

##### 5. Promise.prototype.catch() 原型函数
catch 函数用于捕捉链式调用过程的异常，它底层实现其实是调用 then(null, reject)，then 函数请参考上面解释

##### 6. Promise.prototype.all() 原型函数
all 函数用于多个 Promise 并发执行，所有 Promise 执行完毕并且成功，才会执行 then 的 resolve 回调函数。
resolve 回调函数接收到参数是一个数组，包含了所有 Promise 的返回值。
如果其中一个 Promise 报错，则会直接执行 then 的 reject 回调函数，但其他 Promise 依然会继续执行。

##### 7. Promise.prototype.race() 原型函数
race 函数用于多个 Promise 赛跑执行，只要有一个 Promise 执行完毕并且成功，立即执行 then 的 resolve 回调函数；
只要有一个 Promise 执行失败，立即执行 then 的 reject 回调函数。


### 三，Promise 使用
##### 1. then 链式调用，以及 catch 捕捉异常
```javascript
 imgPm('img0',imgUrl[0])
.then(function(msg){
    console.log(msg)
    // 抛出异常
    // throw new Error('img0 resolve throw new Error');
    return imgPm('img1',imgUrl[1]);
})
.then(function(msg){
    console.log(msg)
    return imgPm('img2',imgUrl[2]);
})
.then(function(msg){
    console.log(msg)
    return imgPm('img3',imgUrl[3]);
})
.then(function(msg){
    console.log(msg);
})
// 捕捉异常，或者接受 Rejected 状态的 Promise
.catch(function(e){
    console.log('catch', e);
});
```

##### 2. Promise.prototype.all 和 race 的使用
```javascript
// 多个 Promise
loadImgAll=[
    imgPm('img0',imgUrl[0]),
    imgPm('img1',imgUrl[1]),
    imgPm('img2',imgUrl[2]),
    imgPm('img3',imgUrl[3])
];

// 并发执行，所有执行完毕，并且都是 Resolved 状态
Promise.all(loadImgAll).then(function(msg) {
    console.log("all done " + msg)
}).catch(function(e) {
    console.log("one error " + e)
});

// 并发执行，只要有一个执行完毕
Promise.race(loadImgAll).then(function(msg) {
    console.log("one done " + msg)
}).catch(function(e) {
    console.log("one error " + e)
});

```

##### 3. [更多测试用例](https://github.com/luckybirdme/blog/blob/master/example/js/test-promise-func.html)


### 四，Promise 原生实现

主要实现 Promise 的状态改变，resolve , reject , then , 以及 catch 函数
```javascript

// 构造函数
function MyPromise(callback){
    var _this = this;
    // 记录 Promise 状态，以及传递参数
    _this.PromiseStatus = 'pending';
    _this.PromiseValue = undefined;
    
    // 实例化 Promise 时，执行的函数
    callback(function(msg){ // 将 Promise 状态 从 pending 变成 resolved
        if(_this.PromiseStatus == 'pending'){
            _this.PromiseStatus = 'resolved';
            _this.PromiseValue = msg;    
        }
    },function(msg){ // 将 Promise 状态 从 pending 变成 rejected
        if(_this.PromiseStatus == 'pending'){
            _this.PromiseStatus = 'rejected';
            _this.PromiseValue = msg;  
        }
    });   
}
// 返回指定状态的 Promise 函数
// 直接对象调用方式，直接追加到 Promise 对象属性
MyPromise.resolve = function(rtn){
    // 如果传入参数是 Promise，则直接返回，有可能是 rejected 状态的 Promise
    if(rtn instanceof MyPromise){
        return rtn;
    // 否则，返回一个状态为 resolved 新的 Promise，参数透传
    }else{
        return new MyPromise(function(resolve,reject){
            resolve(rtn);
        });
    }
    
};
MyPromise.reject = function(msg){
    // 只会返回 rejected 状态的 Promise，参数透传
    return new MyPromise(function(resolve,reject){
        reject(msg);
    });
};

// 链式调用的 then 函数
// 通过 new 构造函数后才能调用，追加到 Promise 的原型
MyPromise.prototype.then = function(resolve,reject){
    var rtn = undefined;
    try{
        // resolved 状态的回调函数
        if(this.PromiseStatus == 'resolved'){
            if(resolve){
                rtn = resolve(this.PromiseValue);    
            }else{
                // 没有 resolved 回调函数，继续透传 resolved 状态的 Promise
                return MyPromise.resolve(this.PromiseValue);
            }
        // rejected 状态的回调函数
        }else if(this.PromiseStatus == 'rejected'){
            if(reject){
                rtn = reject(this.PromiseValue);    
            }else{
                // 没有 rejected 回调函数，继续透传 rejected 状态的 Promise
                return MyPromise.reject(this.PromiseValue);
            }
        }else {
            console.log('PromiseStatus : pending');
            return this;
        }

    // 捕捉回调函数的异常，返回 rejected 状态的 Promise
    }catch(e){
        return MyPromise.reject("Error: "+e.message);
    }

    // 如果回调函数有返回，而且是 Promise，则直接返回
    if(rtn instanceof MyPromise){
        return rtn;
    // 其他情况，返回 resolved 状态的 Promise
    // 比如回调函数返回的是字符串，类数组，k-v对象
    }else{
        return MyPromise.resolve(rtn);
    }
};
// catch 函数，直接调用 then 来实现
MyPromise.prototype.catch = function(reject){
    this.then(null, reject);
};
```

[完整的原生实现例子](https://github.com/luckybirdme/blog/blob/master/example/js/my-promise-func.html)



