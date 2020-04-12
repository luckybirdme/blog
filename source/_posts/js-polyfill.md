---
title: js 兼容性编程 
date: 2020-01-18 20:23:15
tags: JavaScript
---

> 由于不同浏览器实现 js 规范时，会有不同的方法，导致需要进行浏览器兼容性编程。
> 对于不同 js 规范版本之间，有些新特性在旧版本不支持，也需要进行版本兼容性编程。

<!-- more -->

## 一. 浏览器兼容性
### 1. dom 的侦听事件 addEventListener 是 W3 标准，但是 IE8 之前的版本却用 attachEvent 实现了，差异如下：
```js
// IE 8
// 侦听
element.attachEvent("onclick", handler)
// 移除
element.detachEvent('onclick', handler);

// Chrome，符合 W3
// 侦听，useCapture 为 true 表示向下捕捉时响应，false 表示向上冒泡时响应，默认为 false
element.addEventListener('click', handler, useCapture)
// 移除
element.removeEventListener('click', handler);
```

### 2. 根据不同浏览器调用不同方法，通过 if else 进行兼容性编程，

```js

var EventUtil = {
    addHandler : function(element, type, handler){
    	// addEventListener 
        if(element.addEventListener){
            element.addEventListener(type, handler, false);
        // IE8 浏览器
        }else if(element.attachEvent){
            element.attachEvent('on' + type, handler);
        }else{
            element['on' + type] = handler;
        }
    },
    removeHandler : function(element, type, handler){
        if(element.removeEventListener){
            element.removeEventListener(type, handler, false);
        }else if(element.detachEvent){
            element.detachEvent('on' + type, handler);
        }else{
            element['on' + type] = null;
        }
    }
}

var btn = document.getElementById('btn');
var handler = function(){
    console.log('Hello World!');
}
EventUtil.addHandler(btn, 'click', handler);

 ```

## 二. 版本兼容性

### 1. JavaScript 关于 NaN 是比较奇怪的一个值，特点如下：
- 当算术运算返回一个未定义的或无法表示的值时，NaN就产生了。但是，NaN并不一定用于表示某些值超出表示范围的情况。将某些不能强制转换为数值的非数值转换为数值的时候，也会得到NaN。
- 与 JavaScript 中其他的值不同，NaN不能通过相等操作符（== 和 ===）来判断 ，因为 NaN == NaN 和 NaN === NaN 都会返回 false。

#### JavaScript 提供 isNaN() 判断变量是否为 NaN，如果参数不是Number类型， isNaN() 函数会首先尝试将这个参数转换为数值，然后才会对转换后的结果是否是NaN进行判断。通过代码来感受下 isNaN() 函数的奇异

```js
// 正常判断
console.log(isNaN(0/0))
console.log(isNaN(1))

// 无法通过 Number 转换，当成 NaN，返回 true
console.log(isNaN('c')) 
console.log(isNaN(undefined))

```

#### ES6 新增了 Number.isNaN 函数，和全局函数 isNaN() 相比，该方法不会强制将参数转换成数字，只有在参数是真正的数字类型，且值为 NaN 的时候才会返回 true。
```js
// 正常判断
console.log(Number.isNaN(0/0))
console.log(Number.isNaN(1))

// 正常判断，返回 false
console.log(Number.isNaN('c')) 
console.log(Number.isNaN(undefined)) 
```


### 2. 对于尚未支持 ES6 的浏览器，如果不存在此 API，则进行原始实现，这种机制称为 Polyfill 实现。

```js
Number.isNaN = Number.isNaN || function isNaN(input) {
	// 必须为数字类型
	// 利用 NaN == NaN 和 NaN === NaN 返回 false 的特性
    return typeof input === 'number' && input !== input;
}
```

#### 目前 Polyfill 不仅可以兼容不同 JS 版本规范差异，而且还能兼容不同浏览器之间的差异。在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/isNaN) 官网的最后面，一般会提及 Polyfill 的实现，[Balel](https://babeljs.io/docs/en/babel-polyfill#docsNav) 也有对应的 Polyfill 包。