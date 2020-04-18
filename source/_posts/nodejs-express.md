---
title: nodejs Web 框架 express
date: 2020-01-31 16:39:11
tags: nodejs
---

> express 是基于 nodejs 的 Web 框架

<!-- more -->


## 一. express 简介
express 是基于 nodejs 的轻量级 Web 框架，简单易用，功能齐全，包括
- http server 服务器，底层利用 nodejs 的 http 类库，创建 server
- http 请求路由管理，通过数组形式保存路由信息，响应时再遍历具体处理方法
- http 返回内容转换，提供多种类型的返回数据，包括文本，json，file 等
- 中间件注册和使用，跟路由保存到同样的数组里，通过 method = middle 字段区分
- Web 页面的视图渲染，借助第三方插件完成 Web 页面的渲染
- 静态文件处理，也是借助第三方插件，作为中间件形式使用

核心流程图如下

![](/img/2019/express.jpg)

## 二. express 模拟实现
根据 express 的核心功能，简单实现下，以便理解框架
### 1. http server 
通过 nodejs 的 http 类实现，透传 app 作为 http 请求和响应的函数。

```js
function createApplication () {
  // 定义入口函数，初始化操作
  let app = function (req, res) {

  }
  // 定义监听方法
  app.listen = function () {
    // 通过 http 模块创建一个服务器实例
    let server = http.createServer(app); 
    // 将参数列表传入，为实例监听配置项
    server.listen(...arguments); 
  }
}
````

### 2. 路由管理
将所有 http 请求方法，路径，处理方法保存到 routes 数组。
通过 next 方法遍历 routes 数组，匹配对应的方法和路径，找到响应方法。

```js
  // 定义入口函数，初始化操作
  let app = function (req, res) {
    const urlObj = url.parse(req.url, true);
    const pathname = urlObj.pathname
    const method = req.method.toLowerCase();

    let index = 0;
    // 不停地执行 next , 遍历 app.routes
    function next(err){
      // 所有路由不能匹配时报错
      if (index >= app.routes.length) {
        return res.end(`Cannot find ${method} ${pathname}`);
      }
      let route = app.routes[index];
      index++;
      if (pathname === route.path) {
        // 找到请求的具体响应方法，不需要继续执行了，所以传递 next
        route.handler(req, res); 
      } else {
        next();
      }
    }
    // 请求来时，默认调用，然后不停地循环遍历路由数组，
    next();
  }

  app.routes = []; // 定义路由数组

  // 获取所有请求方法，比如常见的 GET/POST/DELETE/PUT ...
  let methods = http.METHODS; 
  methods.forEach(method => {
    method = method.toLocaleLowerCase() 
    app[method] = function (path, handler) {
      let layer = {
        method,
        path,
        handler,
      }
      // 将每一个请求保存到路由数组中
      app.routes.push(layer)
    }
  })
```

### 3. 中间件
中间件也存放到路由数组 routes，格式跟 http 请求一样。
统一通过 next 函数遍历处理，只是做个判断处理。

```js
  let app = function (req, res) {
    const urlObj = url.parse(req.url, true);
    const pathname = urlObj.pathname
    const method = req.method.toLowerCase();

    let index = 0;
    function next(err){
      // 所有路由不能匹配时报错
      if (index >= app.routes.length) {
        return res.end(`Cannot find ${method} ${pathname}`);
      }
      let route = app.routes[index];
      index++;
      // 中间件处理
      if (route.method == "middle"){
        if (pathname === route.path) {
          // 中间件执行完，需要继续，所以需要 next
          route.handler(req,res,next);
        } else {
          next();
        }
      // http 请求处理
      }else{
        if (pathname === route.path) {
          // 找到请求的具体响应方法，不需要继续执行了
          route.handler(req, res); 
        } else {
          next();
        }
      }
    }
    // 请求来时，默认调用，然后不停地循环遍历路由数组，
    next();
  }
  // 中间件函数，类型和路由一样
  app.use = function(path, handler) {
    app.routes.push({
      method:"middle",
      path,
      handler
    });
  }

```

### 4. 错误处理
中间件处理时遇到报错，通过抛出异常来拦截，跳到错误处理中间件

```js
  let app = function (req, res) {
    const urlObj = url.parse(req.url, true);
    const pathname = urlObj.pathname
    const method = req.method.toLowerCase();

    let index = 0;
    function next(err){
      // 所有路由不能匹配时报错
      if (index >= app.routes.length) {
        return res.end(`Cannot find ${method} ${pathname}`);
      }
      let route = app.routes[index];
      index++;
      // 中间件处理
      if (route.method == "middle"){
        if(err){
          // 错误处理中间件有四个参数
          if (route.handler.length == 4) {
            // console.log("错误处理啦", req, res);
            route.handler(err, req, res, next);
          } else { // 没有找到继续向下找错误处理函数 
            next(err);
          }
        }else{
          if (pathname === route.path || route.path === '/') {
            // 中间件执行完，需要继续，所以需要 next
            route.handler(req,res,next);
          } else {
            next();
          }
        }
      // http 请求处理
      }else{
        if(err){
          next(err);
        }else{
          if (pathname === route.path || route.method === 'all') {
            // 找到请求的具体响应方法，不需要继续执行了
            route.handler(req, res); 
          } else {
            next();
          }
        }
      }
    }
    // 请求来时，默认调用，然后不停地循环遍历路由数组，
    next();
  }
```


## 三. 完整代码在这里
### 1. 测试文件 index.js

```js
const express = require('./express.js');
const app = express();
// 中间件匹配所有路径
app.use('/',function (req,res,next) {
  console.log('middle for all req ',req.url);
  next();
});
app.use('/test', function (req,res,next) {
  console.log('only /test',req.url);
  next();
});
app.use('/error', function (req,res,next) {
  console.log('catch error',req.url);
  next('error msg');
});

app.get("/hello",function(req,res){
  res.end("hello");
});
app.post("/world",function(req,res) {
  res.end("world");
});


// 错误中间件，四个参数
app.use('/',function (err, req, res, next) {
  console.log('get error msg',err);
  res.end(err);
});

// 如果以上路由都匹配不了，还有兜底路由
app.all("all",function(req,res) {
  console.log('all for 404',req.url);
  res.end("all for 404")
});


// 监听端口
app.listen(8080,function() {
  // body...
  console.log('run express')
});

```

### 2. 核心文件 express.js

```js
let http = require('http');
let url = require('url');

function createApplication () {
  // 定义入口函数，初始化操作
  let app = function (req, res) {
    const urlObj = url.parse(req.url, true);
    const pathname = urlObj.pathname
    const method = req.method.toLowerCase();

    let index = 0;
    function next(err){
      // 所有路由不能匹配时报错
      if (index >= app.routes.length) {
        return res.end(`Cannot find ${method} ${pathname}`);
      }
      let route = app.routes[index];
      index++;
      // 中间件处理
      if (route.method == "middle"){
        if(err){
          // 错误处理中间件有四个参数
          if (route.handler.length == 4) {
            // console.log("错误处理啦", req, res);
            route.handler(err, req, res, next);
          } else { // 没有找到继续向下找错误处理函数 
            next(err);
          }
        }else{
          if (pathname === route.path || route.path === '/') {
            // 中间件执行完，需要继续，所以需要 next
            route.handler(req,res,next);
          } else {
            next();
          }
        }
      // http 请求处理
      }else{
        if(err){
          next(err);
        }else{
          if (pathname === route.path || route.method === 'all') {
            // 找到请求的具体响应方法，不需要继续执行了
            route.handler(req, res); 
          } else {
            next();
          }
        }
      }
    }
    // 请求来时，默认调用，然后不停地循环遍历路由数组，
    next();
  }


  // 定义监听方法
  app.listen = function () {
    // 通过 http 模块创建一个服务器实例
    let server = http.createServer(app); 
    // 将参数列表传入，为实例监听配置项
    server.listen(...arguments); 
  }

  app.routes = []; // 定义路由数组

  // 获取所有请求方法，比如常见的 GET/POST/DELETE/PUT ...
  let methods = http.METHODS; 
  methods.forEach(method => {
    method = method.toLocaleLowerCase() 
    app[method] = function (path, handler) {
      let layer = {
        method,
        path,
        handler,
      }
      // 将每一个请求保存到路由数组中
      app.routes.push(layer)
    }
  })

  // 中间件函数，类型和路由一样
  app.use = function(path, handler) {
    app.routes.push({
      method:"middle",
      path,
      handler
    });
  }

  // 匹配所有路由，兜底的路由策略
  app.all = function (path, handler) {
    app.routes.push({
      method: "all",
      path,
      handler
    });
  }

  // 返回该函数
  return app
}

module.exports = createApplication;

```