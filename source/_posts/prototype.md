---
title: JS 原型链
date: 2019-09-27 22:59:26
tags: JavaScript
---

> 每个实例对象（ object ）都有一个私有属性（称之为 __proto__ ）指向它的构造函数的原型对象（prototype ）。该原型对象也有一个自己的原型对象( __proto__ ) ，层层向上直到一个对象的原型对象为 null。根据定义，null 没有原型，并作为这个原型链中的最后一个环节。

<!-- more -->


## 一，原型链的查找方法

## 实例代码
```js

function A(){
	this.myName='A';
}
// 原型对象 prototype
A.prototype.showName=function(){
	console.log(this.myName);
};

var a = new A();
// 通过 new 将 A 中的上下文即 this 绑定到 a，从而获取 myName 属性
console.log(a.myName);
// 从原型链 __proto__，获取 A 的原型对象 prototype 的方法 showName
a.showName();
// a 的原型链指向 A 的原型对象
console.log(a.__proto__===A.prototype);
// A 的原型对象也有原型链，指向 Object 的原型对象
console.log(A.prototype.__proto__===Object.prototype);
// Object 的原型对象也有原型链，指向 null
// null 没有原型链 __proto__ 属性，它是 JS 最顶层的对象
console.log(Object.prototype.__proto__===null);
```

![](/example/img/__proto__.png)

## 二，原型链的特点
### 1. 所有的函数都拥有 prototype 属性，用于指向原型对象。
### 2. 实例的原型链 __proto__ 指向构造函数的原型对象 prototype。
### 3. 原型对象 prototype 也有原型链属性 __proto__，指向 Object 的原型对象 prototype。
### 4. Object 的原型对象 prototype 也有原型链属性 __proto__，指向最顶层的 null，null 没有原型链。
### 5. 引用类型变量的构造函数 (new Array() , new Object()) 都是由函数 Function 创建，而 Function 自己创建自己，Function 的原型链指向自己的原型对象。


![](/example/img/prototype.png)


