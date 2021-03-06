---
title: JS new 和 Object.create
date: 2019-09-26 18:49:27
tags: JavaScript
---
> new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例
> Object.create()是 ES5 新增的方法，创建一个新对象，指定新对象的__proto__，同时支持自定义新对象的属性

<!-- more -->

### 一，new
##### 1. new 关键字会进行如下的操作：
- 创建一个空的简单JavaScript对象（即{}）；
- 将新对象的原型链指向构造函数的原型对象 ；
- 执行构造函数，并指定上下文 this 为新对象 ；
- 如果该函数没有返回对象，则返回 this。

##### 2. 用原生 JavaScript 实现 new 的功能：
```javascript

// 简单的实现
function myNew(){
    // 创建一个对象
    var obj = {};
    // 获取输入的构造函数
    var func = Array.prototype.shift.call(arguments);
    // 获取输入的其他参数
    var args = Array.prototype.slice.call(arguments);
    // 将对象的原型链指向构造函数的原型对象，以获得构造函数的原型属性和方法
    obj.__proto__ = func.prototype;
    // 执行构造函数，并将作用域指定为当前 obj 对象，以便获得构造函数自身的属性和方法
    var rtn = func.apply(obj,args);
    // 如果构造函数返回值是自定义的对象，直接返回
    if(typeof rtn === 'object'){
        return rtn
    }
    // 构造函数没有返回，就用使用当前对象 obj
    return obj;
}
                
```

##### 3. [完整的实例代码](/example/js/my-new-func.html)

### 二，Object.create
##### 1. 定义：
创建一个新对象，指定新对象的__proto__ 为第一个输入参数，同时支持自定义新对象的属性，特点如下
- 新对象的原型链指向传入的参数对象；
- 新对象可以自定义自身的属性和方法，也能覆盖参数对象的属性和方法；

##### 2. 用原生 JavaScript 实现 Object.create 的功能
```javascript
// 简单的实现
function myObjectCreate(prototype){
    // 空的构造函数
    function F() {}
    // 构造函数的原型对象指向传入的对象
    F.prototype = prototype;
    // 通过 new 方式，创建新对象，新对象的原型链将会指向 F 构造函数的原型对象，也即传入的对象
    // 返回新对象，它的原型链已经指向传入对象了
    return new F();
}
```

##### 3. Object.create 主要用于继承父类的原型对象的属性和方法，具体例子
```javascript
// 用于继承的父类
function People(){
    this.age = 20;
}
People.prototype.getAge = function() {
    console.log(this.age);
}
// 用于继承的子类
function Student(school){
    this.school = school
}


// Object.create() 将会返回一个原型链指向 People.prototype 的新对象
// 将子类 Student 的原型对象指向此新对象，只继承 People 原型对象 prototype 的属性和方法
Student.prototype = Object.create(People.prototype);
// 避免 Student 原型对象的构造函数指针被更改，加以修正
Student.prototype.constructor = Student;
console.log(People.prototype);
console.log(Student.prototype);
var s1 = new Student('Tsinghua');
s1.age = 22;
console.log(s1);
s1.getAge();

```

##### 4. [完整的实例代码](/example/js/my-object-create-func.html)


### 三，new 和 Object.create 的区别
##### 1. new 更多是为了获取构造函数的自身和原型的所有属性和方法
##### 1. Object.create 更多是为了获取构造函数的原型对象的属性和方法，称为[原型式继承](http://blog.luckybird.me/2020/05/01/javascript-extends/)

