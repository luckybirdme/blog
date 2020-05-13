---
title: JS 继承方法
date: 2020-05-01 18:14:57
tags: JavaScript
---

> 继承

<!-- more -->

### 原型链继承

#### 1. 示例

```js
function MyParent(){
    this.bar = 'bar';
    this.foo = ['foo'];
}

function MyChild() {
}

// 将原型对象指向父类的实例，获得父类的属性
MyChild.prototype = new MyParent();

one_child = new MyChild();
two_child = new MyChild();

console.log('one_child',one_child);
console.log('two_child',two_child);

// 基本类型属性可以单独修改
one_child.bar = 'change bar';
// 引用类型属性修改将会影响所有实例
one_child.foo.push('add foo');

console.log('one_child',one_child);
console.log('two_child',two_child);
```

![](/img/2020/js_extend_func_one.png)


#### 2. 优点
- 实现简单
- 父类新增原型属性，子类所有实例都能访问到

#### 3. 缺点
- 子类定义 prototype 时，只能指向一个父类的实例，无法继承多个父类的属性
- 所有实例共享父类的属性，如果其中一个实例修改了父类的引用类型属性，将会影响所有实例。
- 父类在子类定义时已经执行了构造方法，子类后续通过 new 实例化时，无法给父类的构造函数传递参数。


### 借用构造函数继承

#### 1. 示例

```js
function MyParent(bar){
    this.bar = bar;
}
// 父类原型对象的属性
MyParent.prototype.pp='pp';

function MyMother(foo){
    this.foo = foo;
}
MyMother.prototype.mm='mm';


function MyChild(bar,foo) {
	// 借用父类构造函数，并将子类的 this 指向 父类的构造函数，获得父类构造函数的属性
	// 因此每个实例都单独获得了父类构造函数的属性
	// 但是无法获得父类原型对象的属性
    MyParent.call(this,bar);
    // 可以借用多个父类的构造函数，实现继承多个父类
    MyMother.call(this,foo);
}


one_child = new MyChild('one bar',['one foo']);
two_child = new MyChild('two bar',['two foo']);

console.log('one_child',one_child);
console.log('two_child',two_child);

one_child.bar = 'change bar';
// 修改实例引用类型的属性，不会影响其他实例的属性
one_child.foo.push('add foo');

console.log('one_child',one_child);
console.log('two_child',two_child);

// 无法获得父类原型对象的属性
console.log('one_child.pp',one_child.pp);
console.log('two_child.mm',one_child.mm);
```

![](/img/2020/js_extend_func_two.png)


#### 2. 优点

- 可以借用多个父类的构造函数，继承多个父类的属性
- 实例化子类时，借用父类构造函数，此时可以传递参数给父类的构造函数
- 每个实例单独获得一份父类的属性，修改其中一个实例的属性，都不会影响其他实例

#### 3. 缺点

- 只继承了父类构造函数的属性和方法，无法继承父类原型对象的属性和方法
- 每个子类的实例单独维护一份父类的属性和方法，变得臃肿影响性能，也无法共享属性



### 原型链 + 借用构造函数的组合继承（常用）

#### 1. 示例

```js
function MyParent(bar){
    this.bar = bar;
    this.say_hi = function(){
        console.log('hi');
    }
}
MyParent.prototype.pp='pp';
MyParent.prototype.say_hello=function(){
    console.log('hello');
};


function MyChild(bar) {
    // 实例化子类时，第二次调用父类的构造函数
    MyParent.call(this,bar);
}
// 创建子类的原型对象，第一次调用父类的构造函数
MyChild.prototype = new MyParent(['first time']);

one_child = new MyChild(['one bar']);
two_child = new MyChild(['two bar']);

one_child.bar.push('add bar');

console.log('one_child',one_child);
console.log('two_child',two_child);

console.log('one_child.pp',one_child.pp);

```


![](/img/2020/js_extend_func_three.png)


#### 2. 优点
- 可以借用多个父类的构造函数，继承多个父类的属性
- 实例化子类时，借用父类构造函数，此时可以传递参数给父类的构造函数
- 每个实例单独获得一份父类的属性，修改其中一个实例的属性，都不会影响其他实例
- 继承了父类构造函数的属性和方法，也继承了父类原型对象的属性和方法


#### 3. 缺点
- 执行了两次父类的构造函数：一次是在创建子类原型对象时，另一次是在子类的构造函数内部
- 父类中的实例属性和方法既存在于子类的实例中，又存在于子类的原型对象中，浪费太多内存



### 原型式继承

#### 1. 示例

```js
// 父类的属性和方法
var MyParent={
    'bar':['one bar'],
    'say_hi':function(){
        console.log('say_hi');
    }
}

// 创建对象的方法，用于将 obj 指向原型对象。
// ES6 中通过 Object.create() 实现了以下方法
function createObject(obj){
    function F(){};
    // 指向原型对象
    F.prototype = obj;
    return new F();
}


function MyChild() {

}
// 也可以写成
// MyChild.prototype = Object.create(MyParent);
// 通过两层原型链，获取 MyParent 的属性和方法
MyChild.prototype = createObject(MyParent);


one_child = new MyChild();

one_child.bar.push('add bar');
one_child.say_hi();

console.log('one_child',one_child);

```

![](/img/2020/js_extend_func_four.png)


#### 2. 优点
- 类似于复制一个对象，只是用函数 Object.create() 来包装。

#### 3. 缺点
- 多个实例的引用类型属性指向相同，存在篡改的可能
- 子类实例化时，无法传递参数给父类


### 寄生式继承


#### 1. 示例

```js
// 父类的属性和方法
var MyParent={
    'bar':['one bar'],
    'say_hi':function(){
        console.log('say_hi');
    }
}

// 创建对象的方法，用于将 obj 指向原型对象。
// ES6 中通过 Object.create() 实现了以下方法
function createObject(obj){
    function F(){};
    // 指向原型对象
    F.prototype = obj;
    return new F();
}

// 在原型式的基础上套一个函数，用于增强对象的属性和方法
function extendObject(obj){
    var ex = createObject(obj);
    ex.say_hello=function(){
        console.log('say_hello')
    }
    ex.foo = 'foo';
    return ex;
}

function MyChild() {

}
 
MyChild.prototype = extendObject(MyParent);


one_child = new MyChild();

one_child.bar.push('add bar');
one_child.say_hi();
one_child.say_hello();

console.log(one_child.foo);
console.log('one_child',one_child);

```


![](/img/2020/js_extend_func_five.png)

#### 2. 优点
- 在原型式继承外面套了个函数，增加了对象的属性和方法

#### 3. 缺点
- 多个实例的引用类型属性指向相同，存在篡改的可能
- 子类实例化时，无法传递参数给父类


### 原型链+借用构造函数+寄生式继承（常用）

#### 1. 示例

```js
// 父类的属性和方法
function MyParent(bar){
    this.bar = bar;
    this.say_hi = function(){
        console.log('hi');
    }
}

// 父类原型对象的属性和方法
MyParent.prototype.pp='pp';
MyParent.prototype.say_hello=function(){
    console.log('hello');
};


// 创建对象的方法，用于将 obj 指向原型对象。
// ES6 中通过 Object.create() 实现了以下方法
function createObject(obj){
    function F(){};
    // 指向原型对象
    F.prototype = obj;
    return new F();
}

// 在原型式的基础上套一个函数，用于增强对象的属性和方法
function extendObject(obj){
    var ex = createObject(obj);
    ex.extend_hello=function(){
        console.log('extend_hello')
    }
    ex.foo = 'foo';
    return ex;
}


// 此函数是为了子类获得父类原型对象的属性和方法
function inheritPrototype(MyChild, MyParent){
	// 寄生式继承
    // 获取父类原型对象的属性和方法
    var prototype = extendObject(MyParent.prototype); 
    // 修正原型对象的构造函数，指向子类
    prototype.constructor = MyChild;

    // 原型链继承
    // 将父类原型对象的属性和方法放到子类的原型链上
    MyChild.prototype = prototype;  
}

function MyChild(bar) {
	// 借用构造函数继承
    // 通过借用父类的构造函数，获得父类的属性和方法，并且传递参数
    MyParent.call(this,bar);
}
 

inheritPrototype(MyChild,MyParent);
one_child = new MyChild(['one bar']);
two_child = new MyChild(['two bar']);

one_child.bar.push('add bar');
one_child.say_hi();
one_child.say_hello();
one_child.extend_hello();

console.log(one_child.foo);
console.log('one_child',one_child);
console.log('two_child',two_child);


```

![](/img/2020/js_extend_func_six.png)



#### 2. 优点
- 可以借用多个父类的构造函数，继承多个父类的属性
- 实例化子类时，借用父类构造函数，此时可以传递参数给父类的构造函数
- 每个实例单独获得一份父类的属性，修改其中一个实例的属性，都不会影响其他实例
- 继承了父类构造函数的属性和方法，也继承了父类原型对象的属性和方法
- 只在实例化子类时，执行了一次父类的构造函数
- 父类中的实例属性和方法在原型链中只存在一份

#### 3. 缺点
- 寄生组合继承集合了前面几种继承优点，几乎避免了上面继承方式的所有缺陷，是执行效率最高也是应用面最广的，就是实现的过程相对繁琐。


### 使用 ES6 的 class 概念实现继承

#### 1. 示例

```js
// 父类的属性和方法
class MyParent {
    constructor(bar){
        this.bar = bar;
    }

    show_bar(){
        console.log(this.bar);
    }
}

class MyChild extends MyParent{
    constructor(bar,foo){
        super(bar)
        this.foo = foo
    }
    show_foo(){
        console.log(this.foo);
    }
}


one_child = new MyChild(['one bar'],'one foo');
two_child = new MyChild(['two bar'],'two foo');

one_child.bar.push('add bar');
one_child.show_bar();
one_child.show_foo();

console.log('one_child',one_child);
console.log('two_child',two_child);

```



![](/img/2020/js_extend_func_seven.png)


#### 2. 优点
- 语法简单易懂，操作更方便
- ES5 的继承，实质是先创造子类的实例对象 this，然后再将父类的方法添加到子类的 this 上面
- ES6 的继承机制完全不同，实质是先将父类实例对象的属性和方法，加到父类的 this 上面（所以必须先调用super方法），然后再用子类的构造函数修改父类的 this 指向子类。

#### 3. 缺点
- 需要注意的是，class关键字只是原型的语法糖，JavaScript继承仍然是基于原型实现的。
- 由于是 ES6 新特性，并不是所有的浏览器都支持 class 关键字。