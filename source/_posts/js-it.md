---
title: js 迭代器协议，可迭代协议，生成器对象
date: 2020-01-18 20:23:15
tags: JavaScript
---

> iterator protocol, 迭代器协议
> iterable protocol, 可迭代协议
> Generator object, 生成器对象

<!-- more -->

## iterator protocol, 迭代器协议

#### 1. 定义
- 定义了产生一系列值的标准方式，可能是有限个或者无限个值。
- 当值为有限个时，所有的值都被迭代完毕后，则会返回一个默认返回值。
- 满足迭代器协议的对象，称为迭代器。


#### 2. 实现：通过 next() 方法
- 一个无参数函数，该返回包含 done 和 value 两个属性的对象。
- 如果迭代器可以产生序列中的下一个值，则 done 为 false；如果迭代器已将序列迭代完毕，则 done 为 true
- value 是 done 为 false 时，迭代器返回的任何 JavaScript 值。


#### 3. 示例：手动模拟迭代器协议

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
- 满足可迭代协议的对象，称为迭代对象。

#### 2. 实现：通过 Symbol.iterator 属性返回符合迭代器协议的对象
- 这意味着对象或者它原型链上的某个对象，可通过常量 Symbol.iterator 返回符合迭代器协议的对象。
- Symbol.iterator 返回的是一个迭代器，实现了 next() 方法，因此可迭代对象 iterable，也是属于迭代器 iterator。


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

// 通过 for of 语法测试迭代对象
for(var i of it){
    console.log(i);
}

```


## Generator object, 生成器对象


#### 1. 定义
- function* 这种声明方式定义的函数称为生成器函数 generator function。
- 生成器对象是由 generator function 返回的对象，并且它符合迭代器协议和可迭代协议。

#### 2. 实现：通过 function* 和 yield
- generator function 能在执行时暂停，下次 JS 引擎循环事件读取执行任务时，又能从暂停处继续执行，并且保留现场变量等。
- yield 关键字使 generator function 执行暂停和恢复，它后面的表达式值将会返回给生成器的调用者，类似 return 关键字。
- 生成器保留程序执行状态的特性，可用于逐步生成有规律特定数值，而不需要一次性生成，占用太多内存。



#### 3. 示例

```js

function *genertor(item){
    let value = item;
    let index = 0;
    do{
        yield item[index]
        index=index+1
    } while (index<item.length)
}       

var gen = genertor([1,2,3,4,5]);
var next = null;
do {
    // 生成器满足迭代器协议和可迭代协议，所以可以通过 next 访问下一个值
    next = gen.next();
    console.log(next);
} while (!next.done)


// 利用生成器，生成斐波那契数列
function *fibonacci(n){
    let one=1
    let two=1
    let i = 0
    i=i+1
    yield one
    if (i >= n) {
        return;
    }
    i=i+1
    yield two
    if (i >= n){
        return
    }
    do{
        i=i+1
        last = one + two
        yield last
        one=two
        two=last
    } while(i < n)
}

var fib = fibonacci(10);
var next = null;
do {
    // 不需要一次性生成数列所有元素，每调用一次，生成一个元素
    next = fib.next();
    console.log(next);
} while (!next.done)

```



