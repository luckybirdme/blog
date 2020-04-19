---
title: JS 节流和防抖函数
date: 2019-10-26 14:55:58
tags: JavaScript
---

> 节流，像水龙 1 秒流 1 滴水，固定时间内触发多次事件，只处理一次请求。
> 防抖，手抖点击了多次提交按钮，但最终只会执行最后一次点击提交的事件。

<!-- more -->


### 一，节流
##### 1. 应用场景
- 不停点击鼠标触发 mousedown 事件，但间隔 100 毫秒才执行一次事件。
- 用户不停滑动到页面底部，触发加载更多事件，间隔 1 秒才发起加载请求。


##### 2. 代码实现
- 每次事件都在固定时间后，才真正触发，达到间隔执行的效果。
- 当事件设置在 timeout 后执行时，设置标志位，避免重复提交等待执行的事件。
- 当事件结束后，标志位复位，方可再次提交同样的等待执行事件。

```js
function throttle(fn,interval=300) {
	// 初始化的标志位，可执行
    var running = false;
    console.log('throttle init');
    // 返回闭包函数，用于保存函数的执行状态
    return function () {
    	// 如果存在等待执行的事件，则直接暂停
        if (running) {
            console.log('running');
            return;
        };
        var _this = this;
        var _args = arguments;
        // 设置标志位，避免重复提交事件
        running = true;
        // 设置 timeout 后执行，达到间隔执行的效果
        setTimeout(function(){
            console.log('run start');
            // 传递 this 上下文，确保回调函数是在自己的上下文执行的
            // 传递回调函数的 arguments
            fn.apply(_this,_args);

            // 执行完毕，标志位复位，以便再次提交执行事件
            running = false;
            console.log('run end');
        },interval);
    };
}
```


##### 3. 测试效果，按照固定频率地执行多次
![](/img/2019/throttle.png)




### 二，防抖
##### 1. 应用场景
- window 页面大小变化触发 resize 事件，只需执行一次。
- 用户注册填写邮箱的验证，只需停止输入后进行一次验证。


##### 2. 代码实现
- 设置事件执行前的固定等待时间。
- 如果此时触发相同事件，则把之前的等待事件清除，重新设置等待事件，而且等待时间是固定的等待时间。

```js

function debounce(fn, interval=300) {
	// 初始化，事件执行前的固定等待时间
    var timeout = null;
    console.log('debounce init');
    return function () {
        console.log('wait again');
        // 再次触发相同事件，把之前的等待执行事件清除掉
        clearTimeout(timeout);
        // 重新设置新的等待事件，等待时间是固定的等待时间
        var _this = this;
        var _args = arguments;
        timeout = setTimeout(function(){
            console.log('run start');
            // 传递 this 上下文，确保回调函数是在自己的上下文执行的
            // 传递回调函数的 arguments
            fn.apply(_this, _args);
            console.log('run end');
        }, interval);
    };
}
```


##### 3. 测试效果，只会执行最后一次
![](/img/2019/debounce.png)




### 三，区别
##### 1. 节流，不管触发多少事件，只会间隔固定时间执行一次，有固定频率地执行多次，类似水龙头 1 秒流 1 滴水。
##### 2. 防抖，不管触发多次事件，只会执行最后一次，而且等待固定的时间才执行，类似手抖点了多次，其实只算最后一次。



### [源代码](/example/js/debounce-and-throttle.html)