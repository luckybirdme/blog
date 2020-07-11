---
title: ES6(ECMA2015) 新特性
date: 2020-06-28 22:53:05
tags: JavaScript
---

> ES6

<!-- more -->


## 一. 关于变量
- let 和 const 声明块级作用域变量，而且必须先定义

```js
// 只在 for 内有效
for(let i=0;i<3;i++){
    console.log('for let',i)
}
// 变量 i 无效
console.log('out let',i)
```


- Set 对象允许你存储任何类型的唯一值，无论是原始值或者是对象引用。

```js
let myArray = ["value1", "value2", "value3"];

// 用Set构造器将Array转换为Set
let mySet = new Set(myArray);

mySet.has("value1"); // returns true

// 用...(展开操作符)操作符将Set转换为Array
console.log([...mySet]); // 与myArray完全一致


// 数组去重
// Use to remove duplicate elements from the array 
const numbers = [2,3,4,4,2,3,3,4,4,5,5,6,6,7,5,32,3,4,5]
console.log([...new Set(numbers)]) 
// [2, 3, 4, 5, 6, 7, 32]

```

- Map 对象保存键值对，并且能够记住键的原始插入顺序。任何值(对象或者原始值) 都可以作为一个键或一个值。

描述


分类|Map|	Object
-|-|-
意外的键|	Map 默认情况不包含任何键。只包含显式插入的键。	|一个 Object 有一个原型, 原型链上的键名有可能和你自己在对象上的设置的键名产生冲突。注意: 虽然 ES5 开始可以用 Object.create(null) 来创建一个没有原型的对象，但是这种用法不太常见。
键的类型|一个 Map的键可以是任意值，包括函数、对象或任意基本类型。| 一个Object的键必须是一个 String 或是Symbol。
键的顺序|Map 中的 key 是有序的。因此，当迭代的时候，一个 Map 对象以插入的顺序返回键值。|一个 Object 的键是无序的。注意：自ECMAScript 2015规范以来，对象确实保留了字符串和Symbol键的创建顺序；因此，在只有字符串键的对象上进行迭代将按插入顺序产生键。
Size|	 Map 的键值对个数可以轻易地通过size 属性获取	|Object 的键值对个数只能手动计算
迭代|	Map 是 iterable 的，所以可以直接被迭代。|	迭代一个Object需要以某种方式获取它的键然后才能迭代。
性能	|在频繁增删键值对的场景下表现更好。|在频繁添加和删除键值对的场景下未作出优化。


```js
let myMap = new Map();
let keyString = 'a string';
myMap.set(keyString, "和键'a string'关联的值");
myMap.get(keyString);    // "和键'a string'关联的值"

for (let [key, value] of myMap) {
  console.log(key + " = " + value);
}
// 将会显示两个log。一个是"0 = zero"另一个是"1 = one"
for (let key of myMap.keys()) {
  console.log(key);
}
for (let value of myMap.values()) {
  console.log(value);
}

```


- 可迭代类型 iterable, 包括 Array，Map，Set，String，TypedArray，arguments 对象等等

- for...of语句在可迭代对象上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句

```js
const array1 = ['a', 'b', 'c'];
for (const element of array1) {
  console.log(element);
}
// expected output: "a"
// expected output: "b"
// expected output: "c"


```


- 解构赋值

```js

let [a,b]=[1,2]
// 相当于
let a=1;
let b=2;
```

## 二. 关于参数

- 延展操作符(...args)

```js
function test(...args){
    console.log(args)
}

test(1,2,3)
```



- 函数默认参数

```js
// ES5
function add(x) {
    var x = x || 20;
    return x;
}
console.log(add()); // 20
console.log(add(false)); // 20


//ES6
function add(x = 20) {
    return x ;
}
console.log(add());// 20
console.log(add(false)); // false
```


## 三. 关于函数

- Promise 异步编程

```js
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('foo');
  }, 300);
});

promise1.then((value) => {
  console.log(value);
  // expected output: "foo"
});

```

- 箭头函数

```js

function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this| 正确地指向 p 实例
  }, 1000);
}

var p = new Person();
```


## 四. 关于继承

- 类的支持 class、extend、super

```js
class Square extends Polygon {
  constructor(length) {
    // Here, it calls the parent class' constructor with lengths
    // provided for the Polygon's width and height
    super(length, length);
    // Note: In derived classes, super() must be called before you
    // can use 'this'. Leaving this out will cause a reference error.
    this.name = 'Square';
  }

  get area() {
    return this.height * this.width;
  }
}

```

## 五. 关于模块

- import 用于引入模块，export 用于导出模块。

```js
// nodejs 环境
// module "my-module.js"
function cube(x) {
  return x * x * x;
}
const foo = Math.PI + Math.SQRT2;
var graph = {
    options: {
        color:'white',
        thickness:'2px'
    },
    draw: function() {
        console.log('From graph draw function');
    }
}
export { cube, foo, graph };


// script demo.js
import { cube, foo, graph } from 'my-module.js';
graph.options = {
    color:'blue',
    thickness:'3px'
}; 
graph.draw();
console.log(cube(3)); // 27
console.log(foo);    // 4.555806215962888
```
