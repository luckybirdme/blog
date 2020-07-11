---
title: JS 变量复制
date: 2020-06-12 07:36:40
tags: JavaScript
---

> 变量复制

<!-- more -->


## 一. 变量赋值

1. 基本类型的变量可直接访问和操作对应值，赋值就可以复制一份值到另外一个内存地址中保存，新变量更改时，原变量不会变。


```js
var param1 = 'a';
var param2 = param2;
param2 = 'b';
console.log(param1);
console.log(param2);
//a
//b
```



2. 引用类型的变量则只是保存引用地址，赋值只是复制引用地址，实际上还是共用变量，某一个变量更改时，所有相同引用地址的变量也更改。

```js
var obj1={ name : 'one '};
var obj2=obj1;
obj2.name = 'two';

console.log(obj1);
console.log(obj2);
// {name: "two"}
// {name: "two"}

```

## 二. 变量复制

根据上面变量赋值的特性，基本类型变量的复制，可采用直接赋值的方式；但是对于引用类型变量，则必须通过深复制，或者说是深拷贝。
浅拷贝和深拷贝只针对像 Object, Array 这样的复杂对象的。浅拷贝只拷贝当前对象或者一层对象的属性，而深复制则递归复制了对象所有层级属性，主要区别如下：


1. 新对象修改时，原对象会改变，因此只是复制了对象的引用，属于浅复制。

```js
var obj = { a:1 };
// 对于引用类型的变量，直接赋值，只是引用复制，并没有产生新对象。
var shallowObj = obj;

console.log(obj.a);
console.log(shallowObj.a);
obj.a=4;
console.log(obj.a);
// 属性会改变
console.log(shallowObj.a);

```

2. 新对象第一层属性修改时，原对象不会改变，但是新对象第二层属性修改时，原对象第二层属性也会改变，因为没有进行递归的复制，因此也属于浅复制。

```js

var obj = { a:1, arr: [2,3] };
var shallowObj = shallowCopy(obj);

function shallowCopy(src) {
  var dst = {};
  for (var prop in src) {
	if (src.hasOwnProperty(prop)) {
	  dst[prop] = src[prop];
	}
  }
  return dst;
}
console.log(obj.a);
console.log(shallowObj.a);
obj.a=4;
console.log(obj.a);
// 第一层属性不会改变
console.log(shallowObj.a);


console.log(obj.arr[0]);
console.log(shallowObj.arr[0]);
obj.arr[0]=5;
console.log(obj.arr[0]);
// 第二层属性改变了
console.log(shallowObj.arr[0]);
```

## 三. 浅拷贝的方法
利用特殊的JS方法，返回新对象，原对象不改动，以便浅拷贝第一层属性。


### slice

1. 方法描述
slice() 方法返回一个新的数组对象，这一对象是一个由 begin 和 end 决定的原数组的浅拷贝（包括 begin，不包括end），原始数组不会被改变。
对于字符串，功能和参数都一样。

2. 拷贝规则

- 如果该元素是个对象引用，slice 会拷贝这个对象引用到新的数组里。两个对象引用都引用了同一个对象。如果被引用的对象发生改变，则新的和原来的数组中的这个元素也会发生改变。
- 对于字符串、数字及布尔值来说（不是 String、Number 或者 Boolean 对象），slice 会拷贝这些值到新的数组里。在别的数组里修改这些字符串或数字或是布尔值，将不会影响另一个数组。

3. 使用示例

```js
let a = [0,1,2]
// 复制数组
let b = a.slice()
console.log(a)
console.log(b)
b[0]=5
// a[0] 不变
console.log(a)
// b[0] 会变
console.log(b)

let s = 'abc'
// 复制字符串
let ss = s.slice()
console.log(s)
console.log(ss)
ss='efg'
console.log(s)
console.log(ss)
```

### concat
1. 方法描述
concat() 方法用于合并两个或多个数组。此方法不会更改现有数组，而是返回一个新数组。
对于字符串，功能和参数都一样。

2. 拷贝规则
- 与方法 slice 一样

3. 使用示例

```js
let a = [0,1,2]
let b = a.concat()
console.log(a)
console.log(b)
b[0]=5
console.log(a)
console.log(b)

let s = 'abc'
let ss = s.concat()
console.log(s)
console.log(ss)
ss='efg'
console.log(s)
console.log(ss)
```

### assign

1. 方法描述
Object.assign() 方法用于将所有可枚举属性的值从一个或多个源对象复制到目标对象。它将返回目标对象。JS一切皆对象，数组和字符串也是对象，所以也适用此方法。

2. 拷贝规则
Object.assign 方法只会拷贝源对象自身的并且可枚举的属性到目标对象。该方法使用源对象的[[Get]]和目标对象的[[Set]]，所以它会调用相关 getter 和 setter。因此，它分配属性，而不仅仅是复制或定义新的属性。String类型和 Symbol 类型的对象都会被拷贝。


3. 使用示例

```js
let obj = {'key1':1,'key2':2}
let copy = Object.assign({},obj)
console.log(obj)
console.log(copy)
copy['key1']=5
console.log(obj)
console.log(copy)


let a = [0,1,2]
let b = Object.assign([],a)
console.log(a)
console.log(b)
b[0]=5
console.log(a)
console.log(b)

```




## 三. 深拷贝的方法

利用 JSON 序列化和反序列化的过程，达到深拷贝对象的效果

### stringify
1. 方法描述
JSON.stringify(value[, replacer [, space]]) 方法将一个 JavaScript 值（对象或者数组）转换为一个 JSON 字符串，如果指定了 replacer 是一个函数，则可以选择性地替换值，或者如果指定了 replacer 是一个数组，则可选择性地仅包含数组指定的属性。

2. 序列化规则

- 转换值如果有 toJSON() 方法，该方法定义什么值将被序列化。
- 非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中。
- 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
- undefined、任意的函数以及 symbol 值，在序列化过程中会被忽略。
- 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。
- 所有以 symbol 为属性键的属性都会被完全忽略掉，即便 replacer 参数中强制指定包含了它们。
- Date 日期调用了 toJSON() 将其转换为了 string 字符串（同Date.toISOString()），因此会被当做字符串处理。
- NaN 和 Infinity 格式的数值及 null 都会被当做 null。

### parse
1. 方法描述
JSON.parse(text[, reviver]) 方法用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。提供可选的 reviver 函数用以在返回之前对所得到的对象执行变换(操作)。

2. 反序列化规则
若传入的字符串不符合 JSON 规范，则会抛出 SyntaxError 异常。

3. 使用示例

```js

let a={'one':1,'two':[0,1]};
console.log(a);
let b = JSON.parse(JSON.stringify(a));
console.log(b);
b['one']=4;
b['two'][0]=5;
console.log(a);
console.log(b);

```


## 四. 原生实现深拷贝

为了避免误修改引用类型的变量，需要一个深克隆的功能；针对不同的类型，深克隆方法也不一样
1. 数组，层层遍历，直到属性值为基本类型的变量

```javascript
var newArr = [];
for (var i = 0; i < input.length ; i++) {
	// 递归克隆
	newArr[i] = myDeepClone(input[i]);
}
return newArr;

```

2. 对象，跟数组类似的方式，但要考虑到枚举属性和继承属性

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

3. 日期，直接用 new

```javascript
return new Date(input);

```

4. 正则，也直接用 new

```javascript
return new RegExp(input);

```

5. 函数，克隆函数自身的属性，以及原型对象的属性，避免构造函数被误修改

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

6. 其他类型，比如 Symbol，后续再研究 


[示例代码](/example/js/deep-clone-func.html)



## 四，业界的克隆方法


- [loadsh](https://lodash.com/docs/4.17.15#cloneDeep)，一个实用的工具库，提供 array、number、objects、string 等数据类型更高效快捷的操作方法

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

- [immutable](https://github.com/immutable-js/immutable-js)，高性能的数据类库，复制对象时，只更改对应节点以及父节点，子节点则共享，以减少遍历。在 React 中判断是否更新数据渲染模板时，对于复杂对象是非常有用的。

```javascript
// 深克隆
var immutable= new Object().AsImmutable();
var newObj = immutable.GetCopy();
```

- [jQuery](https://api.jquery.com/jquery.extend/)，几乎是每个网站的标配类库，当然少不了深克隆的方法。

```javascript
var obj1 = {'a':'one','b':'two'};
var obj2 = {'c':'three'};
// 浅克隆
// 第一个参数是目标对象，第二参数起，是将要复制的对象，可以接收第三个参数，第 n 个对象
var obj2=$.extend({},obj1,obj2);
// 深克隆
// 第一个参数改为 true，后续参数跟浅克隆一样，第二是目标对象，第三个起将要复制的队形
// 第一个参数不能传递 false，官网文档说不支持
var obj2=$.extend(true,{},obj1,obj2);
```