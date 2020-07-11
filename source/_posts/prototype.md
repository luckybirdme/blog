---
title: JS 原型对象和原型链
date: 2019-09-27 22:59:26
tags: JavaScript
---

> 在典型的面向对象的语言中，如PHP，都存在类 class 概念，类就是对象的模板，对象就是类的实例。
> 在JavaScript语言体系中，是不存在类 class 的概念的，继承特性不是基于 class ，而是通过原型链 prototype chains 实现的。
> 在ES6中提供了更接近传统语言的写法，引入了类 class 概念，但它仍然基于原型的，只是一种语法糖，让原型对象的写法更加清晰、更像面向对象编程的语法而已。

<!-- more -->

## 一. 类和继承
#### 1. 典型面向对象语言的继承方式，以PHP为例
```php

class MyParent 
{
 
    public $parent_value = 'parent_value';

    public function __construct($class_name)
    {
        $this->class_name = $class_name;
    }

    public function show_name(){
        echo $this->class_name.PHP_EOL;
    }

}

// 继承父类的属性和方法
class MyChild extends MyParent
{
    
    public function __construct($class_name)
    {
        $this->class_name = $class_name;
    }
}


$test_mychild = new MyChild('test_mychild');
$test_mychild->show_name();
echo $test_mychild->parent_value.PHP_EOL;

```


#### 2. JavaScript 继承的方式

```javascript

function MyParent(class_name){

    this.parent_val = 'parent_val';

    this.class_name = class_name;

    this.show_name = function(){
        console.log(this.class_name);
    }
}

function MyChild(class_name) {
   this.class_name = class_name
}
// 继承父类的属性和方法
MyChild.prototype = new MyParent();

test_child = new MyChild('test_child');
test_child.show_name();
console.log(test_child.parent_val);

```

从上述代码可以看出

- JavaScript 的继承没有典型面向对象的 class，extends 的概念
- JavaScript 通过 prototype 属性继承了父级函数的共有方法和属性

这里只是 JavaScript 的继承方法之一，还有很多的[继承方式](http://blog.luckybird.me/2020/05/01/javascript-extends/)，但是都是基于 prototype 属性来实现的。这个 prototype 就是将要说的原型对象。


## 二. 原型对象
#### 1. 定义
- 所有函数都有一个特别的属性 prototype，这个属性是一个指针，指向一个对象，称为原型对象。
而这个对象的用途是包含特定类型的所有实例共享的属性和方法。
- 只要创建了一个新函数，会根据特定的规则为该函数创建一个prototype 属性指向原型对象。在默认情况下，所有原型对象都会自动获得一个constructor（构造函数）属性，这个属性是一个指向 prototype 属性所在函数的指针。
- 访问实例化对象的属性时，首先从构造函数自身作用域查找，如果没有找到，则从构造函数的 prototype 上查找。实例化对象是通过 \_\_proto\_\_ 这个属性访问到构造函数的 prototype 上的属性。

通过图片来理解

![](/img/2020/js_extend_one.png)

通过代码来理解

```javascript
    

function MyParent(class_name){

    this.parent_val = 'parent_val';

    this.class_name = class_name;

    this.show_name = function(){
        console.log(this.class_name);
    }
}

// 原型对象上的属性和方法将会在所有实例中共享
// 直接在原型对象上添加属性和方法
MyParent.prototype.foo='foo';
MyParent.prototype.say_hi = function(){
    console.log('hello',this.class_name);
};

console.log('MyParent.prototype',MyParent.prototype);

one_parent = new MyParent('one_parent');
console.log('one_parent',one_parent);
one_parent.show_name();
console.log(one_parent.class_name);
one_parent.say_hi();
console.log(one_parent.foo);

two_parent = new MyParent('two_parent');
console.log(two_parent);
two_parent.show_name();
console.log(two_parent.class_name);
two_parent.say_hi();
console.log(two_parent.foo);


```

MyParent 函数 和 one_parent 对象

![](/img/2020/js_extend_two.png)



## 三. 原型链
#### 1. 规则
- 访问实例化对象的属性时，首先从构造函数自身作用域查找，如果没有找到，则从构造函数的 prototype 上查找。实例化对象是通过 \_\_proto\_\_ 这个属性访问到构造函数的 prototype 上的属性。
- prototype 称为原型对象，本身也对象，所以也有\_\_proto\_\_ 指针，可以继续从 \_\_proto\_\_ 链查找，这个链就是原型链。
- 原型链倒数第二个元素是 Object.prototype，Object.prototype 的原型链 \_\_proto\_\_  指向 null，而 null 是没有 \_\_proto\_\_  指针的，所以原型链查找到此结束。
- \_\_proto\_\_ 其实是隐式原型链 implicit prototype link，JavaScript中任意对象都有一个内置属性[[prototype]] 在ES5之前没有标准的方法访问这个内置属性，但是大多数浏览器都支持通过\_\_proto\_\_来访问。ES5中有了这个内置属性标准的Get方法 Object.getPrototypeOf()。
- 不合适通过 \_\_proto\_\_ 来直接修改原型链的属性，不仅性能低下，而且会引发各种意向不到的问题，比如 constructor 构造函数指向混乱，官方也不推荐这样使用。
- JavaScript 中一切皆对象，因此 Function 关键字定义函数也是对象，也有 \_\_proto\_\_ 的原型链。引用类型变量的构造函数 (new Array(), new Object()) 都是由函数 Function 创建，因此 Array, Object 的原型链 \_\_proto\_\_ 指向 Function.prototype，而 Function 自己创建自己，Function 的原型链\_\_proto\_\_指向自己的原型对象 Function.prototype。

还是图片来感受

![](/img/2020/js_extend_three.png)

![](/img/2020/prototype.jpg)


```javascript
function MyParent(){
}

one_parent = new MyParent();
console.log('one_parent',one_parent);

console.log('one_parent.__proto__===MyParent.prototype',one_parent.__proto__===MyParent.prototype);

console.log('MyParent.__proto__===Function.prototype',MyParent.__proto__===Function.prototype);
console.log('Array.__proto__===Function.prototype',Array.__proto__===Function.prototype);
console.log('Object.__proto__===Function.prototype',Object.__proto__===Function.prototype);
console.log('Function.__proto__===Function.prototype',Function.__proto__===Function.prototype);


console.log('MyParent.prototype.__proto__===Object.prototype',MyParent.prototype.__proto__===Object.prototype);
console.log('Function.prototype.__proto__===Object.prototype',Function.prototype.__proto__===Object.prototype);

console.log('Object.prototype.__proto__==null',Object.prototype.__proto__==null);

```



![](/img/2020/js_extend_five.png)



## 四. 创建对象和生成原型链

#### 1. 通过 new 关键字，执行构造函数创建对象

```js
function Graph() {
  this.vertices = [];
  this.edges = [];
}

Graph.prototype = {
  addVertex: function(v){
    this.vertices.push(v);
  }
};


var g = new Graph();
// g 是生成的对象，他的自身属性有 'vertices' 和 'edges'。
// 在 g 被实例化时，g.[[Prototype]] 指向了 Graph.prototype。

```

#### 2. 使用语法结构创建的对象

```js

var o = {a: 1};

// o 这个对象继承了 Object.prototype 上面的所有属性
// o 自身没有名为 hasOwnProperty 的属性
// hasOwnProperty 是 Object.prototype 的属性
// 因此 o 继承了 Object.prototype 的 hasOwnProperty
// Object.prototype 的原型为 null
// 原型链如下:
// o ---> Object.prototype ---> null

var a = ["yo", "whadup", "?"];

// 数组都继承于 Array.prototype 
// (Array.prototype 中包含 indexOf, forEach 等方法)
// 原型链如下:
// a ---> Array.prototype ---> Object.prototype ---> null

```


#### 3. 使用 Object.create 创建的对象，ECMAScript 5 中引入了一个新方法：Object.create()。

```js

var a = {a: 1}; 
// a ---> Object.prototype ---> null

var b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (继承而来)

var c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

var d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); 
// undefined, 因为d没有继承Object.prototype

```

#### 4. 使用 class 关键字创建的对象，ECMAScript6 引入了一套新的关键字用来实现 class。使用基于类语言的开发人员会对这些结构感到熟悉，但它们是不同的。JavaScript 仍然基于原型。这些新的关键字包括 class, constructor，static，extends 和 super。

```js

"use strict";

class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

class Square extends Polygon {
  constructor(sideLength) {
    super(sideLength, sideLength);
  }
  get area() {
    return this.height * this.width;
  }
  set sideLength(newLength) {
    this.height = newLength;
    this.width = newLength;
  }
}

var square = new Square(2);

```