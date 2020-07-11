---
title: JS 变量类型
date: 2019-09-29 06:52:34
tags: JavaScript
---

> 基本类型和引用类型

<!-- more -->

## 1. 基本类型：
变量通过 typeof 即可判断类型，直接保存变量值，主要包括：
- Undefined，已经定义但没有赋值，比如 var a; 此时 a=undefined;
- Null，这是一个对象，也是 JavaScript 最顶层的对象，typeof Null == 'object'
- Boolean，两种值，true or false
- Number，包括整型和浮点型
- String，字符串类，比如'abc'

```javascript
var a;
undefined
typeof a;
"undefined"
typeof null
"object"
typeof 3.0
"number"
typeof 'abc'
"string"
typeof true
"boolean"
```


## 2. 引用类型：
引用类型变量只是指针，指向具体内容变量内容。
用 typeof 判断并不准确，数组也是返回对象，instanceof 也不准确，因为都是对象的实例；最准确的方式通过 toString，返回最原始的变量类型
```javascript
// 数组也是对象类型
typeof []
"object"
typeof {}
"object"

// 数组也是对象的实例
new Array() instanceof Object
true
new Object() instanceof Object
true

// 最准确的方式
Object.prototype.toString.call([])
"[object Array]"
Object.prototype.toString.call({})
"[object Object]"
Object.prototype.toString.call(function(){})
"[object Function]"
Object.prototype.toString.call(new Date())
"[object Date]"
Object.prototype.toString.call(1)
"[object Number]"
Object.prototype.toString.call('a')
"[object String]"
Object.prototype.toString.call(true)
"[object Boolean]"
Object.prototype.toString.call(undefined)
"[object Undefined]"
Object.prototype.toString.call(null)
"[object Null]"
Object.prototype.toString.call(new RegExp('/[0-9]/'))
"[object RegExp]"
```