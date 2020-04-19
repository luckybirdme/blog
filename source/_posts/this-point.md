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
### 1. 直接调用普通函数，函数内部 this 指向 window
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
### 2. 多层对象嵌套，谁调用对象，对象的 this 指向谁
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


## 三，ES6新特性

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
