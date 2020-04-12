---
title: js iterator
date: 2020-01-18 20:23:15
tags: JavaScript
---

> iterator 迭代器，是一个接口，作用是便于遍历数据的所有元素。 

<!-- more -->

### 1. JS 迭代器协议具体参考 MDN 的文档 [iterator](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)

### 2. JS 的 String, Array, TypedArray, Map and Set 都是可迭代的对象，原型对象都有一个 [Symbol.iterator] 方法，即迭代器。

```js
let str = '123';

let i = str[Symbol.iterator]();

console.log(i.next());
console.log(i.next());
console.log(i.next());
console.log(i.next()); 
/*
{ value: '1', done: false }
{ value: '2', done: false }
{ value: '3', done: false }
{ value: undefined, done: true }
*/

```

### 3. JS 中实现了迭代协议的数据，可通过 for of 和 [...] 访问

```js
let str='123';
console.log([...str]);
for(let i of str){
  console.log(i)
}
```

### 4. 迭代器的简单实现

```js
function makeIterator(array){
    var nextIndex = 0;
    
    return {
       next: function(){
           return nextIndex < array.length ?
               {value: array[nextIndex++], done: false} :
               {value: undefined, done: true};
       }
    };
}

var it = makeIterator(['a', 'b', 'c']);

console.log(it.next()); 
console.log(it.next()); 
console.log(it.next());
console.log(it.next());
```

### 5. 通过 [generator function](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator) 和 yield (https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield) 实现

```js
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
    yield 'a';
    yield 'b';
    yield 'c';
};
console.log([...myIterable]);
var it = myIterable[Symbol.iterator]()
console.log(it.next()); 
console.log(it.next()); 
console.log(it.next());
console.log(it.next());

```