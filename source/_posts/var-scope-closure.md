---
title: JS 作用域和闭包
date: 2019-09-27 08:09:03
tags: JavaScript
---

> 作用域负责收集并维护当前范围内的所有声明的变量，确定当前执行的代码对这些变量的访问权限，这个范围就叫作用域。
> 函数在每次创建时生成闭包，闭包也是一个函数，这个闭包函数允许其他外部函数访问当前作用域中的变量。

## 一，作用域
### 1. 定义
作用域负责收集并维护当前范围内的所有声明的变量，确定当前执行的代码对这些变量的访问权限，这个范围就叫作用域。

### 2. 作用
可见，作用域最大的用处就是隔离变量，不同作用域下同名变量不会有冲突。



## 二. 变量作用域
### 1. 变量作用域主要分为
- 全局作用域，在 window 下定义的变量，可被所有作用域访问。
- 局部作用域，在函数对象内部定义的变量，只可在函数内部访问，除非使用闭包。
- 块级作用域，这是 ES6 新增的特性，只在 if，for，while 等语句块里的作用域，通过 let const 变量使用。

##### 下面看具体代码

```js
// window 下的全局作用域变量
var color = "blue";
function changeColor(){
    // changeColor 函数作用域的变量
    var anotherColor = "red";
    function swapColors(){
        // swapColors 函数作用域的变量
        var tempColor = 'green';

        // 这里可以访问 color、anotherColor 和 tempColor
        console.log('swapColors access color',color);
        console.log('swapColors access anotherColor',anotherColor);
        console.log('swapColors access tempColor',tempColor);
        
    }
    
    swapColors();

    // 这里可以访问 color 和 anotherColor，但不能访问 tempColor
    console.log('changeColor access color',color);
    console.log('changeColor access anotherColor',anotherColor);
    console.log('changeColor not access tempColor',typeof tempColor);

    
}

changeColor();


// 这里只能访问 color
console.log('window access color',color);
console.log('window not access anotherColor',typeof anotherColor);
console.log('window not access tempColor',typeof tempColor);


```


##### 以上的访问关系链也成为作用域链，如下图所示


![](http://qiniucdn.luckybird.me/blog/img/2019/scope.png)



##### 块级作用域示例代码

```js
// var 变量定义没有块级作用域，如果不在函数对象内，默认就是全局变量
for(var i=0;i<3;i++){
    console.log('inner',i);
}
// 依然输出 3
console.log('outer',i);


// let 和 const 有块级作用域
for(let j=0;j<3;j++){
    console.log('inner',j);
}
// 输出 undefined
console.log('outer',typeof j);

// const 是常量变量，一经定义和赋值，不能再修改
if(true){
    const a = 'a';
    console.log('inner',a);
}
console.log('outer',typeof a);

```


### 2. 使用变量的注意事项
##### 为了更规范编写 JS 代码，以下列举的注意事项都尽量避免。

- 如果变量不使用 var let const 定义，默认是全局作用域变量，即使是在函数内部定义的。


```js
function test(){
    // 没有使用关键字定义，默认全局作用域变量
    not_use_var='not_use_var';
}
test();
console.log(not_use_var);

```

- var 变量可在定义语句前就使用，因为 JS 代码编译时会将变量定义先放入内存，默认值为 undefined，这种情况也叫变量提升，但是对于 let const 不存在变量提升，必须先定义再使用，否则报错。


```js
console.log(use_var_before_defined);
var use_var_before_defined = 'use_var_before_defined';
console.log(use_var_before_defined);

// Cannot access 'use_let_before_defined' before initialization
console.log(use_let_before_defined)
let use_let_before_defined = 'use_let_before_defined';
console.log(use_let_before_defined);
```

- var 变量可重复定义使用，但是 let const 不可以重复定义


```js
var a='1';
// 1
var a='2';
// 2
let b='1';
// 1
let b='2';
//Uncaught SyntaxError: Identifier 'b' has already been declared
```

- const 是常量变量，定义为基本类型变量后就不能修改，但是对于引用型变量，是可以修改的


```js
const c=1;
c=2
// Uncaught TypeError: Assignment to constant variable.
const d={};
d['a']='a';
// {a: "a"}
```

### [测试变量作用域的源代码](https://github.com/luckybirdme/blog/blob/master/example/js/var-scope.html)


## 二. 闭包
### 1. 定义
闭包是由函数以及创建该函数的作用域组合而成，这个闭包函数保存了当前作用域所能访问的局部变量，并常驻于内存中。

### 2. 作用
外部函数可通过闭包访问函数内部的局部变量，可以理解成，闭包是不同作用域之间的访问桥梁。


## 三，闭包的使用
### 1. 闭包使用的具体案例
- 返回闭包函数，以便访问函数内的变量


```js
// 通过闭包访问函数内的局部变量
var person=(function(){
    //
    var myName="jack";
    return function(){
        return myName;
    }
})(); // 最后一个括号的意思是，函数立即执行，函数执行后返回的是一个闭包函数

// 无法直接访问函数的局部变量
console.log(typeof myName);

// 执行闭包函数
// 获取函数局部变量
console.log(person());
```

- 将变量的值常驻内存中，以便下次访问


```js
// 通过闭包缓存数据
var Cache=(function(){
    var cache={};
    return function(name){
        // 检查是否存在缓存
        if(name in cache){
            console.log(name,'had cache',cache[name]);
            return cache[name];
        }

        var temp=new Object(name);
        // 将缓存保存到函数内的局部变量 cache 中
        // 因为闭包的特性，cache 会常驻内存中，以便下次访问
        cache[name]=temp;
        console.log(name,'create cache',cache[name]);
        return temp;
    }
    
})();

Cache('one');
Cache('two');
Cache('one');
```

- 封装成模块化开发，避免污染全局变量


```js
var Car=(function(){
    var _this = this;
    _this.color=[];
    _this.number=0;
    return {
        getColor:function(){
            return _this.color;
        },
        setColor:function(color){
            _this.color.push(color);
        },
        getNumber:function(){
            return _this.number;
        },
        addNumber:function(){
            _this.number++;
        }
    }

})();

Car.setColor('red');
Car.setColor('green');
console.log(Car.getColor());

Car.addNumber();
Car.addNumber();
console.log(Car.getNumber());
```

- 其他应用场景


```js
// 绑定并排元素的场景
var oneLi=document.querySelectorAll("#one li");
for (var i  = 0; i < oneLi.length; i++) {
    oneLi[i].onclick=function(){
        // 将会全部输出 3，因为 i 只保存了最后一个值
        console.log('number',i);
    };
  
}
var twoLi=document.querySelectorAll("#two li");
for (var j  = 0; j < twoLi.length; j++) {
    // 使用闭包函数
    twoLi[j].onclick=(function(j){
        return function(){
            console.log('number',j);
        }
    })(j); // 立即执行，产生闭包函数，并返回闭包作为 onclick 触发时执行的函数，此时 j 变量已经常驻于内存中
  
}
```


### 2. 闭包使用的注意事项
- 如果闭包使用过多，会导致内存消耗过高，导致页面性能下降。
- 函数执行完后，函数的局部变量没有释放，容易造成内存泄露。

##### 解决方法
闭包不再使用时，要及时释放，将获得闭包的变量赋值为 null，JavaScript 的垃圾回收机制会根据对象是否被引用来释放内存。
```js
function func() {
    //数组占用了很大的内存空间
    var arr = new Array[100000]; 
    // 闭包保存数组长度到内存里
    return function () {
        console.log(arr.length);
        return arr.length;
    }
}
var f = func();
f()
//让闭包函数成为垃圾对象，回收闭包
f = null 
```


### [测试闭包的源代码](https://github.com/luckybirdme/blog/blob/master/example/js/scope-closure.html)







