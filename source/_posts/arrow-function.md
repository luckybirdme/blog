---
title: 箭头函数
date: 2020-06-28 22:51:31
tags: JavaScript
---

> x => return x+1

<!-- more -->


箭头函数表达式更适用于那些本来需要匿名函数的地方

1. 不会创建自己的this,它只会从自己的作用域链的上一层继承 this

- ES5

```js
function Person() {
  var that = this;
  that.age = 0;

  setInterval(function growUp() {
    // 默认是 window 
    console.log(this)
    //  回调引用的是`that`变量, 其值是预期的对象. 
    that.age++;
  }, 1000);
}

var p = new Person();

```



- ES6

```js

function Person(){
  this.age = 0;

  setInterval(() => {
    console.log(this)
    // this 正确地指向 Person 实例
    this.age++; 
  }, 1000);
}

var p = new Person();

```

由于箭头函数没有自己的this指针，通过 call() 或 apply() 方法调用一个函数时，只能传递参数，不能绑定this，他们的第一个参数会被忽略。这种现象对于bind方法同样成立。

2. 箭头函数不绑定Arguments 对象。因此，在本示例中，arguments只是引用了封闭作用域内的arguments：

```js
var arguments = [1, 2, 3];
var arr = () => arguments[0];
```

在大多数情况下，使用剩余参数是相较使用arguments对象的更好选择。

```js
function foo(arg1,arg2) { 
    var f = (...args) => args[1]; 
    return f(arg1,arg2); 
} 
foo(1,2);  //2
```


3. 箭头函数不能用作构造器，和 new一起用会抛出错误。

```js
var Foo = () => {};
var foo = new Foo(); // TypeError: Foo is not a constructor
```

4. 箭头函数没有prototype属性。

```js
var Foo = () => {};
console.log(Foo.prototype); // undefined
```

5. yield 关键字通常不能在箭头函数中使用（除非是嵌套在允许使用的函数内）。因此，箭头函数不能用作函数生成器。 
