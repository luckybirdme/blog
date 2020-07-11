---
title: let 和 const 变量
date: 2020-06-28 22:50:05
tags: JavaScript
---

> let
> const

<!-- more -->


1. ES5 声明变量只有关键字 var，存在变量提升的问题，可以在定义之前使用，这是因为 JS 引擎会先解析定义语句 var param，并赋值 undefined

```js
console.log('param',param)
var param='1';
console.log('param',param)
/*
test.html:14 param undefined
test.html:16 param 1
*/
```

在for循环结束后，var 的变量依然有效，容易埋下隐患

```js
for(let i=0;i<3;i++){
    console.log('for',i)
}
console.log('out',i)
/*
test.html:15 for 0
test.html:15 for 1
test.html:15 for 2
test.html:17 out 3
*/
```


2. ES6 新增了声明变量关键字 let，它必须先定义才能使用，否则直接报错，而且 let 只在 for 循环内的块级作用域有效，可以避免以上问题

```js
console.log('param',param)
let param='1';
console.log('param',param)

/*
test.html:15 Uncaught ReferenceError: Cannot access 'param' before initialization
    at test.html:15
*/

for(let i=0;i<3;i++){
    console.log('for let',i)
}
console.log('out let',i)
/*
test.html:15 for let 0
test.html:15 for let 1
test.html:15 for let 2
test.html:17 Uncaught ReferenceError: i is not defined
    at test.html:17
*/
```

3. const 变量跟 let 类似，只不过它主要用来定义常量，定义的时候必须赋值初始化，而且定义后无法修改，但是对于引用型变量，还是可以修改具体数据的

```js

// 会报错，只定义没有初始化
cons t;
// 正常
const a = 1;
// 会报错，基本数据类型不能在初始化后更改
a = 2; 


const b = {};
// 正常，因为引用类型的指针没有发生变化
b.one = 1;
// 会报错
b = []
```


