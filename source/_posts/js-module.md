---
title: JS 的模块机制
date: 2020-01-18 20:23:15
tags: JavaScript
---

> - JS 初期属于简单的浏览器脚本语言，简单的代码逻辑即可实现理想的效果。
> - 随着 Web 和 node 发展，JS 实现的功能越来越复杂，由于缺少类和模块的概念，代码组织也越来越难。
> - 通过对象和闭包，JS 能模拟命名空间的效果，达到简单的模块化管理，但是缺少标准的规范，而且功能也较弱。
> - 由此产生了几种 JS 模块规范，主要应用于服务端的 CommonJS，浏览器端的 AMD、CMD，以及兼容两端的 ES6 模块规范。

<!-- more -->

## 一. CommonJS
### 1. 规范：
- 一个单独的文件就是一个模块，每一个模块都是一个单独的作用域，在该模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性。
- 模块的加载是同步的而且可以加载多次，但在第一次加载后就会被缓存，后面再次被加载时会直接从缓存中读取。

### 2. 使用：
- 通过exports和modul.exports来暴露模块中的内容
- 通过require来加载模块

```javascript
// 模块文件 math.js
function add(a, b) {
  return a + b;
}
//需要向外暴露的变量、函数
module.exports = {
  add
}

//引入自定义的模块时，参数包含路径，可省略.js
var math = require('./math');
// 调用模块的具体函数
math.add(1, 2)

```

require 具体流程
1. 解析文件的路径
2. 判断缓存是否有这个文件
3. 如果有，则直接返回文件对应的模型
4. 如果没有，则读取文件内容，实例化模型
5. 保存模型到缓存中
6. 返回这个文件模型

模拟实现 nodejs 的 require
```js

let path = require('path')
let fs = require('fs')
let vm = require('vm')

// 定义模型
function Module(fullPath) {
    this.fullPath = fullPath;
    this.exports = {};
}
 
Module._extentions = ['.js', '.json'];
// 模型缓存
Module._cache = {};
 
// 解析文件路径
Module._resovleFilename = function(relativePath){
    let p = path.resolve(__dirname, relativePath);
    let ext = path.extname(p);
    if (ext && Module._extentions[ext]) {
        // 写的是全路径
    } else {
        // 拼接
        for (let i = 0; i < Module._extentions.length; i++) {
            let fullPath = p + Module._extentions[i];
            try{
                fs.accessSync(fullPath);
                return fullPath
            }catch(e){
                console.log(e)
            }
        }
    }
}

// 加载模型
Module.prototype.load = function(){
    // js加闭包，json直接当对象
    let ext = path.extname(this.fullPath)
    // 读取和执行文件内容
    Module._extentions[ext](this);
}
Module.wrapper = ["(function(exports, module, require){" , "\n})"]
Module.wrap = function(script) {
    return Module.wrapper[0] + script +  Module.wrapper[1];
}
Module._extentions['.js'] = function(module){
    // 读取文件内容
    let codeText = fs.readFileSync(module.fullPath);
    // 构造模型函数的内容
    let fnStr = Module.wrap(codeText)
    // 执行模型函数里的内容
    let fn = vm.runInThisContext(fnStr);
    fn.call(module.exports, module.exports, module, req)
}
Module._extentions['.json'] = function(module){
    // json 文件直接读取，然后解析
    let codeText = fs.readFileSync(module.fullPath);
    module.exports = JSON.parse(codeText)
}
function req(p) {
    // 搞出绝对路径
    let fullPath = Module._resovleFilename(p);
    // 拿到绝对路径去缓存里找
    if(Module._cache[fullPath]) {
      return Module._cache[fullPath];
    }
    // 没有缓存，进行加载
    let module = new Module(fullPath);
    module.load();
    // 放到缓存里
    Module._cache[fullPath] = module.exports;
    return module.exports
}
 
let test = req('./test')
console.log(test);

```


### 3. 特点
- module.exports只能输出一个对象，且后面的会覆盖上面的。
- exports相当于 var exports = module.exports，所以不能直接赋值，否则切断了exports与module.exports的联系
- require是同步的，所以会阻塞js执行，在服务端取决于磁盘文件读取速度，但在浏览器端则跟网络关系大，所以更合适在服务端使用。
- 模块既是对象，加载的是整个模块对象，即将所有的接口全部加载进来，但是运行时才会确定具体的接口。
- 输出是值的拷贝，即原来模块中的值改变不会影响已经加载的该值。

### 4. 实现
- nodejs
- webpack

## 二. AMD

### 1. 规范：
- 采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

### 2. 使用：
- 由于不是JavaScript原生支持，使用AMD规范进行页面开发需要用到对应的库函数，也就是require.js
- 语法格式：define(<模块名称>, [依赖数组], 回调函数)

```javascript
//math.js定义一个模块:
define('math', ['jquery'], function (jquery) { //引入jQuery模块
  return {
    add: function (x, y) {
      return x + y;
    }
  };
});

// 导入和使用：
require(['math'], function (math) {
  math.add(1, 2)
})

```

### 3. 特点
- 实现js文件的异步加载，避免网页失去响应；
- 管理模块之间的依赖性，便于代码的编写和维护。


### 4. 实现
- requireJS
- webpack

## 三. ES6 模块
### 1. 规范：
- ES6中无需引入别的js文件，ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。
- ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。

### 2. 使用：
- 使用export关键字来导出模块，使用import关键字引用模块
- export default，为模块指定默认输出.对应导入模块import时，不需要使用大括号。

```javascript
//math.js
var num = 0;
var add = function (a, b) {
  return a + b;
};
export { num, add };

//导入
import { num, add } from './math';
function test(ele) {
  ele.textContent = add(1 + num);
}

```

### 3. 特点
- export 可以输出多个，输出方式为 {},export default 输出一个默认值。
- 解析阶段确定对外输出的接口，解析阶段生成接口，可进行静态分析。
- 输出的是值的引用，值改变，引用也改变，即原来模块中的值改变则该加载的值也改变。
- this 指向undefined
- 当前浏览器并没有完全实现了 ES6 的所有语法，还需要 babel-loader 支持 ES6 模块

### 4. 实现
- nodejs
- webpack


