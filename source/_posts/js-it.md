---
title: js Iteration protocols 
date: 2020-01-18 20:23:15
tags: JavaScript
---

> iterator protocol, 迭代器协议
> iterable protocol, 可迭代协议
> Generator object, 生成器对象，实现了迭代器协议和可迭代协议


<!-- more -->

## iterator protocol, 迭代器协议

#### 1. 定义
定义了产生一系列值的标准方式，可能是有限个或者无限个值。当值为有限个时，所有的值都被迭代完毕后，则会返回一个默认返回值。


#### 2. 实现：通过 next() 方法
- 一个无参数函数，该返回包含 done 和 value 两个属性的对象。
- 如果迭代器可以产生序列中的下一个值，则 done 为 false；如果迭代器已将序列迭代完毕，则 done 为 true
- value 是 done 为 false 时，迭代器返回的任何 JavaScript 值。


#### 3. 示例：手动模拟迭代器

```js

var it = {
    'value':['a','b','c'],
    'index':0,
    'next':function(){
        var value = '';
        if(this.index<this.value.length){
            value = this.value[this.index];
            this.index = this.index+1;
            return {
                'done':false,
                'value':value
            }
        }
        return {
            'done':true,
            'value':''
        }
    }
}

console.log(it.next());
console.log(it.next());
console.log(it.next());
console.log(it.next());
console.log(it.next());

```


## iterable protocol, 可迭代协议

#### 1. 定义
- 允许 JavaScript 对象定义它们的迭代行为，例如，在一个 for..of 结构中，哪些值可以被遍历到
- 一些内置类型同时是内置可迭代对象，有默认的迭代行为，比如 Array 或者 Map，而其他内置类型则不是，比如 Object

#### 2. 实现：通过 @@iterator 方法
- 这意味着对象或者它原型链上的某个对象，必须有一个键为 @@iterator 的属性，可通过常量 Symbol.iterator 访问该属性
- Symbol.iterator 是一个迭代器，实现了 next() 方法，因此可迭代对象 iterable，也是属于迭代器 iterator。


#### 3. 示例：手动模拟可迭代协议

```js

var it = {
    'value':['a','b','c'],
    'index':0,
    'next':function(){
        var value = '';
        if(this.index<this.value.length){
            value = this.value[this.index];
            this.index = this.index+1;
            return {
                'done':false,
                'value':value
            }
        }
        return {
            'done':true,
            'value':''
        }
    }
}

it[Symbol.iterator]=function(){
    return this;
};


// 由于迭代器只能执行一次，所以需要注释掉这里
/*console.log(it.next());
console.log(it.next());
console.log(it.next());
console.log(it.next());
console.log(it.next());*/

// 通过 for of 语法测试迭代器
for(var i of it){
    console.log(i);
}

```


## Generator object, 生成器对象


#### 1. 定义
- function* 这种声明方式定义的函数称为生成器函数 generator function。
- 生成器对象是由 generator function 返回的对象，并且它符合可迭代协议和迭代器协议。

#### 2. 实现：通过 function* 和 yield
- generator function 能在执行时暂停，下次 JS 引擎循环事件读取执行任务时，又能从暂停处继续执行，并且保留现场变量等。
- yield 关键字使 generator function 执行暂停和恢复，它后面的表达式值将会返回给生成器的调用者，类似 return 关键字。


#### 3. 示例

```js
// 官网案例
function *gen(){
    yield 10;
    x=yield 'foo';
    yield x;
}

var gen_obj=gen();
// 使用可迭代协议访问
console.log(gen_obj.next());// 执行 yield 10，返回 10
console.log(gen_obj.next());// 执行 yield 'foo'，返回 'foo'
console.log(gen_obj.next(100));// 将 100 赋给上一条 yield 'foo' 的左值，即执行 x=100，返回 100
console.log(gen_obj.next());// 执行完毕，value 为 undefined，done 为 true

```



