---
title: JS 变量和克隆
date: 2019-09-29 06:52:34
tags: JavaScript
---

> JavaScript 变量包括基本类型和引用类型，基本类型的变量可直接访问和操作对应值，赋值就可以克隆一份值到另外一个内存地址中保存。
> 引用类型的变量则只是保存引用地址，赋值只是克隆引用地址，实际上还是共用变量，变量更改时，所有相同引用地址的变量也更改。

<!-- more -->

### 一，变量类型简介和判断
##### 1. 基本变量，通过 typeof 来判断：
- Undefined，已经定义但没有赋值，比如 var a; 此时 a=undefined;
- Null，这是一个对象，也是 JavaScript 最顶层的对象，typeof Null == 'object'
- Boolean，两种值，true or false
- Number，包括整型和浮点型
- String，字符串类，比如'abc'

```javascript
var a;
undefined
typeof a;
"undefined"
typeof null
"object"
typeof 3.0
"number"
typeof 'abc'
"string"
typeof true
"boolean"
```


##### 2. 引用类型：
引用类型变量用 typeof 判断并不准确，数组也是返回对象，instanceof 也不准确，因为都是对象的实例；最准确的方式通过 toString，返回最原始的变量类型
```javascript
// 数组也是对象类型
typeof []
"object"
typeof {}
"object"

// 数组也是对象的实例
new Array() instanceof Object
true
new Object() instanceof Object
true

// 最准确的方式
Object.prototype.toString.call([])
"[object Array]"
Object.prototype.toString.call({})
"[object Object]"
Object.prototype.toString.call(function(){})
"[object Function]"
Object.prototype.toString.call(new Date())
"[object Date]"
Object.prototype.toString.call(1)
"[object Number]"
Object.prototype.toString.call('a')
"[object String]"
Object.prototype.toString.call(true)
"[object Boolean]"
Object.prototype.toString.call(undefined)
"[object Undefined]"
Object.prototype.toString.call(null)
"[object Null]"
Object.prototype.toString.call(new RegExp('/[0-9]/'))
"[object RegExp]"
```


### 二，简单的变量浅克隆
浅克隆和深克隆主要区别在于，克隆对象更变后，是否影响原有变量，对于基本类型的克隆都是深克隆。
```javascript
var param1 = 'a';
var param2 = param2;
param2 = 'b';
console.log(param1);
console.log(param2);
//a
//b
```
### 三，变量深克隆的原生实现
对于引用类型，如果只是复制引用地址，属于浅克隆，克隆对象改变时，原对象也改变
```javascript
var obj1={ name : 'one '};
var obj2=obj1;
obj2.name = 'two';

console.log(obj1);
console.log(obj2);
// {name: "two"}
// {name: "two"}

```

为了避免误修改引用类型的变量，需要一个深克隆的功能；针对不同的类型，深克隆方法也不一样
- 数组，层层遍历，直到属性值为基本类型的变量

```javascript
var newArr = [];
for (var i = 0; i < input.length ; i++) {
	// 递归克隆
	newArr[i] = myDeepClone(input[i]);
}
return newArr;

```

- 对象，跟数组类似的方式，但要考虑到枚举属性和继承属性

```javascript
// 只克隆可枚举属性
for(var one in input){
	// 只克隆自身属性，忽略原型链的继承属性
	if(input.hasOwnProperty(one)){
		// 递归克隆
		newObj[one] = myDeepClone(input[one])
	}
}
return newObj;

```

- 日期，直接用 new

```javascript
return new Date(input);

```

- 正则，也直接用 new

```javascript
return new RegExp(input);

```

- 函数，克隆函数自身的属性，以及原型对象的属性，避免构造函数被误修改

```javascript
// 克隆函数自身属性
var temp = function temporary() {
	return input.apply(this, arguments); 
};          

// 克隆函数原型对象
for(var key in input.prototype) {
	// 避免构造函数被修改
	if ( key!='constructor' && input.prototype.hasOwnProperty(key)) {
		// 递归克隆
		temp.prototype[key] = myDeepClone(input.prototype[key]);
	}
}
return temp;

```

- 其他类型，比如 Symbol，后续再研究 


##### [示例代码](/example/js/deep-clone-func.html)

### 四，业界的克隆方法
##### 1. [jQuery](https://api.jquery.com/jquery.extend/)，几乎是每个网站的标配类库，当然少不了深克隆的方法。

```javascript
var obj1 = {'a':'one','b':'two'};
// 浅克隆
var obj2=$.extend(false,{},obj1);
// 深克隆
var obj2=$.extend(true,{},obj1);
```

##### 2. [loadsh](https://lodash.com/docs/4.17.15#cloneDeep)，一个实用的工具库，提供 array、number、objects、string 等数据类型更高效快捷的操作方法

```javascript
// 浅克隆
var objects = [{ 'a': 1 }, { 'b': 2 }];
var shallow = loadsh.clone(objects);
console.log(shallow[0] === objects[0]);

// 深克隆
var func = function(){
	console.log('hello');
};
var deep = loadsh.cloneDeep(func);
console.log(deep === func);
```

##### 3. [immutable](https://github.com/immutable-js/immutable-js)，高性能的数据类库，复制对象时，只更改对应节点以及父节点，子节点则共享，以减少遍历。在 React 中判断是否更新数据渲染模板时，对于复杂对象是非常有用的。
```javascript
// 深克隆
var immutable= new Object().AsImmutable();
var newObj = immutable.GetCopy();
```

##### 4. [Object.assign](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)，ES6 中的新特性，但只是浅克隆对象的属性值，无法进行深克隆属性值为引用的对象。
```javascript
const obj = { a: 1 };
const copy = Object.assign({}, obj);
console.log(copy); // { a: 1 }
```