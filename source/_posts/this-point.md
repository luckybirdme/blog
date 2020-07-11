---
title: JS this 指向
date: 2019-09-27 07:51:54
tags: JavaScript
---

> JavaScript 的作用域是定义就决定，但是执行时上文下即 this 会变动。
> 运行时会在指定上下文运行，不同 this 指向，会获取到不同对象的属性。
> 默认情况下，谁调用对象，那么对象内的 this 就指向谁，除非绑定其他对象。

<!-- more -->


## 一，默认情况
### 1. 直接调用普通函数，相当于执行 window.func()，所以函数内部 this 指向 window
```js
// 非严格模式下，默认 this 指向 window
// 比如普通函数调用

var defaultVar='defaultVar';
function testFunc(){
	console.log(this);
	console.log(this.defaultVar)
}

// 函数调用相当于 window.testFunc()，window 下有全局变量 defaultVar
testFunc();
```
### 2. 多层对象嵌套，谁调用对象，对象内部的 this 就指向谁
```js
// 多层对象调用
var testObj={
	myName:'mike',
	getName:function(){
		return this.myName
	}
};

// 输出 mike，因为 testObj 调用了getName，this 绑定了 testObj，拥有 myName 属性
console.log(testObj.getName());
var getName = testObj.getName;
// 输出 undefined，因为函数调用相当于 window.getName()，this 绑定了 window，而 window 没有 myName 属性
console.log(getName());

```


## 二，特殊情况
### 1. 通过 apply，call，bind 等方法更改对象的 this 指向
```js
// 更改 this 指向
var changeThisObj={
    myThis:'oneThis',
    getThis:function(){
        return this.myThis;
    }
}
var testThis={
    myThis:'twoThis'
}
// 此时 this 指向 changeThisObj
var oneThis = changeThisObj.getThis();
// 输出 oneThis
console.log(oneThis);

// 通过 apply 更改了 this 指向，现在指向 testThis
var twoThis = changeThisObj.getThis.apply(testThis);
// 输出 twoThis
console.log(twoThis);
```


### 2. 通过 new 执行构造函数，会将构造函数内部的 this 指向一个新对象，并返回。
```js
// 构造函数的情况
function MyConstuctor(){
    console.log(this);
    this.funcName='constuctor';
}

// 直接调用构造函数，跟普通函数调用一样，this 指向 window
MyConstuctor();

// 通过 new 执行构造函数，会指向构造函数实例化的新对象
var myFunc=new MyConstuctor();
console.log(myFunc.funcName);
```

##### new 底层实质也是通过类似 apply，call，bind 的函数更改了 this 指向


## 三，模拟实现 call 和 apply

### 1. fn.call(obj,param) 是将 fn 函数内部执行时的 this 指向了 obj，并传递了 param 参数

- 原函数例子
```js
var foo={
    val:'foo'
}
function bar(one,two){
    console.log('one:',one,';two:',two);
    console.log(this.val);
}
// 直接执行会打印 undefined
// 因为在当前环境，默认是 window.bar(), 此时 this 就指向 window, window 没有 val 属性
bar()

// 将 bar 函数执行时的 this 指向 foo, 此时打印 foo
bar.call(foo,'one','two');
```

- 模拟实现原理：将 bar 函数设置为 foo 的属性，那么 foo.bar() 时，根据谁调用，this 就指向谁，此时 bar 函数内部执行时的 this 指向了 foo

```js
var foo={
    val:'foo',
    // 对象增加属性，即函数，那么通过该对象执行函数时，函数内部的 this 就会指向该对象了
    bar: function(one,two){
        console.log('one:',one,';two:',two);
        console.log(this.val);
    }
}

// 将 bar 函数执行时的 this 指向 foo, 此时打印 foo
// 相当于 call 的效果了
foo.bar('one','two');
```

- 模拟实现的代码

```js

var foo={
    val:'foo'
}

// 参数跟原始 call 函数一样
// 第一个就是 this 指向的新对象
// 后面分开的一个个方法参数
Function.prototype.myCall=function(){
    // 获取新对象，即新的上下文
    // 默认情况下为 window
    var context =arguments[0]||window;

    // 为新对象 context 增加一个属性方法 fn ，fn 是调用 myCall 函数的方法
    // 后面执行 context.fn() 时，fn 方法内部的 this 就会指向 context ，达到 call 的效果
    // 这里的 this 是调用 myCall 函数的方法的上下文 this，跟 fn 方法内部的 this (指向 context)不一样，
    context.fn=this

    // 获取其他参数，采用字符串形式保存，以便后续通过 eval 调用
    var args=[];
    // 第二参数可能为空，需要特殊处理
    if (arguments.length>1){
        // 第一个参数是 context，所以从第二参数开始
        for(var i = 1; i < arguments.length; i++){
            args.push('arguments['+i+']');
        }        
    }

    var js='context.fn()';
    // 如果存在第二参数
    if(args.length>0){
        // toString() 会变成 arguments[1],arguments[2]
        js='context.fn('+args.toString()+')';
    }
    // eval 会把字符串当做 js 代码来执行
    console.log('js',js);
    var res = eval(js);

    // 删除之前新增的属性方法，避免修改对象
    delete context.fn
    return res;

}

function bar(one,two){
    console.log(arguments)
    console.log(this.val)
}
bar.call(foo,'one','two');
bar.myCall(foo,'one','two');
```


### 2. fn.apply(obj,[]) 跟 fn.call() 只是参数不一样，所以实现起来大同小异

```js
// 参数跟 apply 一样
// 第一个就是 this 指向的新对象
// 后面是类数组参数，比如 arguments
Function.prototype.myApply=function(){
    // 获取新对象，即新的上下文
    // 默认情况下为 window
    var context =arguments[0]||window;

    // 为新对象 context 增加一个属性方法 fn ，fn 是调用 myCall 函数的方法
    // 后面执行 context.fn() 时，fn 方法内部的 this 就会指向 context ，达到 call 的效果
    // 这里的 this 是调用 myCall 函数的方法的上下文 this，跟 fn 方法内部的 this (指向 context)不一样，
    context.fn=this

    // 获取其他参数，采用字符串形式保存，以便后续通过 eval 调用
    var args=[];
    // 第二参数可能为空，需要特殊处理
    if (arguments.length>1){
        // 这里跟 call 不一样，第一个参数是 context，第二参数是类数组，所以从 0 开始遍历类数组
        for(var i = 0; i < arguments[1].length; i++){
            args.push('arguments[1]['+i+']');
        }    
    }
    
    var js='context.fn()';
    // 如果存在第二参数
    if(args.length>0){
        js='context.fn('+args.toString()+')';
    }
    // eval 会把字符串当做 js 代码来执行
    console.log('js',js);
    var res = eval(js);

    // 删除之前新增的属性方法，避免修改对象
    delete context.fn
    return res;

}


function test(){
    console.log(arguments);
    console.log(this.val);
}
test.apply(foo,['one','two']);
test.myApply(foo,['one','two']);

```


## 四，ES6新特性

### 1. 非严格模式下，默认 this 指向 window；但是严格模式下，this 默认是 undefined
```js
// 严格模式，默认 this 不是指向 window，而是 undefined
var stictVar = 'stictVar';
function testStrict(){
	// 使用严格模式
    'use strict';
    // 输出 undefined
    console.log(this);
    // 这里会报错
    // console.log(this.stictVar);
}
testStrict();

```

### 2. 箭头函数
- 没有 this 的，只能从外层函数寻找，所以在定义时就决定了 this 指向。
- 因为没有自己的 this，所以也不能作为构造函数，new 会直接报错。
- 没有 arguments 参数，无参数格式 ()=>{}，单参数 x=>{}，多参数 (...args)=>{}。

```js
// 箭头函数，默认 this 指向外层对象
var testArrow={
    myName:'arrow func',
    showNameByDefault:function(){
        // setTimeout 回调函数是匿名函数，默认 this 指向 window
        setTimeout(function(){
            // 输出 undefined
            console.log(this.myName);
        });
    },
    showNameByArrow:function(){
        // setTimeout 回调函数是箭头函数，默认 this 指向外层对象，即 testArrow
        setTimeout(()=>{
            // 输出 arrow func
            console.log(this.myName);
        });
    }
};

testArrow.showNameByDefault();
testArrow.showNameByArrow();

```

### [源代码](/example/js/this-point.html)
