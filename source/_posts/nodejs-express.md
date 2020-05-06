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
- 中间件注册和使用，跟路由保存到同样的数组里，通过 method 类型来区分
- Web 页面的视图渲染，借助第三方插件完成 Web 页面的渲染
- 静态文件处理，也是借助第三方插件，作为中间件形式使用

核心流程图如下

![](/img/2019/express.jpg)

## 二. express 核心流程

- 框架启动文件，通过 node app.js 启动 express 框架，此时会启动一个 http server，等待处理请求

```js
// app.js
// 加载 express 框架
const express = require('express')
// 初始化框架配置
const app = express()
const port = 3000

// 添加中间件处理函数
app.use('/', function(req, res, next){
  console.log('middleare 1')
  // 将会去执行路由 handle 函数
  next()
  // handle 函数已经执行完请求
  console.log('middleare 2')
})

// 添加中间件处理函数，可以增加多个，但是必须通过 next() 函数串联起来
app.use('/next/func', function(req, res, next){
  console.log('middleare 1 next')
  // 将会去执行路由 handle 函数
  next()
  // handle 函数已经执行完请求
  console.log('middleare 2 next')
},function(req, res, next){
  console.log('middleare 3 next')
  // 将会去执行路由 handle 函数
  next()
  // handle 函数已经执行完请求
  console.log('middleare 4 next')
})

// 添加路径处理函数
app.get('/', function(req, res){
  console.log('handle 1')
  res.send('Hello World!')
  console.log('handle 2')
})

// 添加路径处理函数，可以增加多个，但是必须通过 next() 函数串联起来
app.get('/next/func', function (req, res, next) {
  console.log('one handle func')
  // 调用下一个处理函数
  next()
}, function (req, res) {
  console.log('two handle func')
  res.send('Hello from B!')
})


// 侦听端口，等待请求到来
app.listen(port, () => console.log(`Example app listening on port ${port}!`))

```

- express 类库入口文件只是简单地返回 express 对象

```js
// node_modules/express/index.js
// 导出 express 对象
module.exports = require('./lib/express');

```

- express 对象返回 app 函数，是处理 http 请求的核心函数，所有请求都会在这里处理。

```js
// node_modules/express/lib/express.js
exports = module.exports = createApplication;

// 初始化 app 函数并返回
function createApplication() {
  
  // 创建 app 函数，将会在 http 请求到来时，执行该函数
  var app = function(req, res, next) {
    console.log('handle 3');
    // 所有 http 请求将会在这里处理
    app.handle(req, res, next);
    console.log('handle 4')
  };

  // 继承 nodejs 的事件驱动类库
  mixin(app, EventEmitter.prototype, false);
  // 继承 application 类，包括常用函数，比如 app.use(), app.get(), app.handle() 等
  mixin(app, proto, false);

  // expose the prototype that will get set on requests
  app.request = Object.create(req, {
    app: { configurable: true, enumerable: true, writable: true, value: app }
  })

  // expose the prototype that will get set on responses
  app.response = Object.create(res, {
    app: { configurable: true, enumerable: true, writable: true, value: app }
  })

  // 初始化各种配置
  app.init();
  // 返回 app 函数，将会在 http 请求到来时，执行该函数
  return app;
}

```

- app.init 函数会进行初始化各种配置

```js
// node_modules/express/lib/application.js

// 初始化配置，比如缓存，渲染引擎，设置等
app.init = function init() {
  this.cache = {};
  this.engines = {};
  this.settings = {};

  // 默认设置
  this.defaultConfiguration();
};


```

- application.js 文件不仅包括了 app.init 初始化函数，还有添加中间件的函数 app.use

```js
// app.js
/*
app.use('/', function(req, res, next){
  console.log('middleare 1')
  next()
  console.log('middleare 2')
})

*/

// 对应框架底层函数
// node_modules/express/lib/application.js
app.use = function use(fn) {
  
  // 初始化路由，将会通过数组形式存放中间件
  // lazy 的意思只会初始化一次，后续会复用
  // setup router
  this.lazyrouter();
  var router = this._router;

  fns.forEach(function (fn) {
    // 如果是中间件处理函数是自定义的，即不是 app 的子对象，则直接加到路由数组中
    // non-express app
    if (!fn || !fn.handle || !fn.set) {
      return router.use(path, fn);
    }
  }, this);

  return this;
};


```

- router.use 函数会保存中间件处理函数，它是通过数组形式保存的，支持一个中间件对应多个处理函数

```js
// app.js
/*
// 添加中间件处理函数，可以增加多个，但是必须通过 next() 函数串联起来
app.use('/next/func', function(req, res, next){
  console.log('middleare 1 next')
  // 将会去执行路由 handle 函数
  next()
  // handle 函数已经执行完请求
  console.log('middleare 2 next')
},function(req, res, next){
  console.log('middleare 3 next')
  // 将会去执行路由 handle 函数
  next()
  // handle 函数已经执行完请求
  console.log('middleare 4 next')
})
*/

// node_modules/express/lib/router/index.js
// 添加中间件到路由数组
proto.use = function use(fn) {
  
  var callbacks = flatten(slice.call(arguments, offset));

  // 支持一个中间件对应多个处理函数
  for (var i = 0; i < callbacks.length; i++) {
    var fn = callbacks[i];

    // add the middleware
    debug('use %o %s', path, fn.name || '<anonymous>')

    // layer 可以理解成处理层，一层就是一个处理方法
    // 所有请求将会通过这一层层进行匹配，匹配成功就会调用此处理函数
    var layer = new Layer(path, {
      sensitive: this.caseSensitive,
      strict: false,
      end: false
    }, fn);

    // 默认 route 属性为 undefined，用于跟路径处理函数区分开来
    layer.route = undefined;
    // 路由数组 this.stack，将会存放所有中间件处理函数，以及路径处理函数
    this.stack.push(layer);
  }

  return this;
};

```

- app.get(), app.post() 等方式绑定的路径处理函数，是通过 router.route() 添加到路由数组 this.stack 的。

```js
// app.js
/*
app.get('/', function(req, res){
  console.log('handle 1')
  res.send('Hello World!')
  console.log('handle 2')
})
*/

// 对应框架底层函数
// node_modules/express/lib/application.js
// 获取所有请求方法，比如常见的 GET/POST/DELETE/PUT ...
var methods = require('methods');
// 遍历所有 GET/POST 等方法，绑定处理函数
methods.forEach(function(method){
  app[method] = function(path){

    // 初始化路由，将会通过数组形式存放中间件
    this.lazyrouter();
    // 调用 router.route() 方法，把路径处理函数当做中间件一样存放到路由数组 this.stack
    var route = this._router.route(path);
    // 调用 route[method] 将一个路径下的多个处理函数单独保存另外一个数组 this.stack 
    route[method].apply(route, slice.call(arguments, 1));
    return this;
  };
});

```

- 此时 router 是将路径处理函数当做中间件来存放，通过设置 layer.route 属性来区分，而且路径处理函数统一使用 route.dispath 来分发一个路径下的多个处理函数

```js

// node_modules/express/lib/router/index.js
// 添加路径处理函数到路由数组
proto.route = function route(path) {
  // 实例化路径对象 route，用于匹配请求的路径
  var route = new Route(path);
  var layer = new Layer(path, {
    sensitive: this.caseSensitive,
    strict: this.strict,
    end: true
  // 路径分发函数，一个路径下可能有多个处理函数，通过此方法分发
  }, route.dispatch.bind(route));

  // 设置了 route 参数，标记为路径处理函数，用于跟中间件处理函数区分
  layer.route = route;
  // 将路径处理函数也放到路由数组中
  // 这里的 this.stack 是 router 路由数组的，存放中间件和路径处理函数
  this.stack.push(layer);
  return route;
};

```

- 路径处理函数实际是通过 route[method] 方法来统一绑定，而且另外创建了一个 this.stack 数组来存放路径处理函数，因为一个路径下可能有多个路径处理函数。

```js
// app.js
/*
// 添加路径处理函数，可以增加多个，但是必须通过 next() 函数串联起来
app.get('/next/func', function (req, res, next) {
  console.log('one handle func')
  // 调用下一个处理函数
  next()
}, function (req, res) {
  console.log('two handle func')
  res.send('Hello from B!')
})
*/

// node_modules/express/lib/router/route.js
// 添加路径处理函数到另外一个数组，一个路径下有多个处理函数
methods.forEach(function(method){
  Route.prototype[method] = function(){
    var handles = flatten(slice.call(arguments));

    // 一个路径对应多个处理函数
    for (var i = 0; i < handles.length; i++) {
      var handle = handles[i];

      debug('%s %o', method, this.path)

      // 绑定路径处理函数到 layer
      var layer = Layer('/', {}, handle);
      layer.method = method;
      this.methods[method] = true;

      // route 单独创建了一个数组 this.stack , 专门存放一个路径下的多个路径处理函数
      // 这里的 this.stack 是 route 路径数组的，一个路径下的多个处理函数
      this.stack.push(layer);
    }

    return this;
  };
});

```

- 当 http 请求到来时，将会执行 express 初始化时返回 app 函数，此函数将会执行 app.handle() 函数

```js
// node_modules/express/lib/application.js

app.handle = function handle(req, res, callback) {

  // 加载路由，包括路由数组中存放的处理函数
  var router = this._router;

  // final handler
  var done = callback || finalhandler(req, res, {
    env: this.get('env'),
    onerror: logerror.bind(this)
  });

  // no routes
  if (!router) {
    debug('no routes defined on app');
    done();
    return;
  }

  // 执行路由数组中的处理函数
  router.handle(req, res, done);
};

```

- router.handle() 会通过 next() 方法遍历路由数组，匹配到对应处理函数，然后判断是中间件还是路径处理函数，最后执行

```js
// node_modules/express/lib/router/index.js
proto.handle = function handle(req, res, out) {
  var self = this;

  debug('dispatching %s %s', req.method, req.url);

  // 路由数组，包括中间件和路径处理函数
  // middleware and routes
  var stack = self.stack;

  // 第一次手动执行 next() ，后续将由中间件和路径处理函数自动调用
  // 如果想暂停路由数组遍历，那不执行该函数即可
  // 此方式非常合适在请求执行前，通过中间件做安全过滤，如果异常直接返回，不执行 next()
  next();

  function next(err) {
    
    // 遍历结束了， 还是没有找到处理函数，抛出异常
    // no more matching layers
    if (idx >= stack.length) {
      setImmediate(done, layerError);
      return;
    }

    // find next matching layer
    var layer;
    var match;
    var route;

    while (match !== true && idx < stack.length) {
      layer = stack[idx++];

      // 进行路由匹配
      match = matchLayer(layer, path);
      route = layer.route;

      // 中间件不需要后续处理
      if (!route) {
        // process non-route handlers normally
        continue;
      }

      // 判断路径处理函数是否匹配
      var method = req.method;
      // 需要进行一些特殊转换，比如 GET => get，然后再判断 
      var has_method = route._handles_method(method);

    }


    // 如果是路径处理函数，透传到 req 中去，以便处理方法里面使用这些信息
    // store route for dispatch on change
    if (route) {
      req.route = route;
    }

    // this should be done for the layer
    self.process_params(layer, paramcalled, req, res, function (err) {
      if (err) {
        return next(layerError || err);
      }

      // 路径处理函数
      if (route) {
        return layer.handle_request(req, res, next);
      }

      // 中间件处理函数
      trim_prefix(layer, layerError, layerPath, path);
    });
  }

  // 中间件处理函数支持匹配到具体路径才执行
  // 比如 /, 则所有路径都执行该中间件
  // 比如 /test, 则只有请求路径是 /test 才执行该中间件
  function trim_prefix(layer, layerError, layerPath, path) {
    
    debug('%s %s : %s', layer.name, layerPath, req.originalUrl);

    if (layerError) {
      layer.handle_error(layerError, req, res, next);
    } else {
      // 中间件处理函数
      layer.handle_request(req, res, next);
    }
  }
};

```

- layer.handle_request() 将会执行具体的处理函数，这些处理函数在 express 初始化和启动时，就已经绑定了

```js
// node_modules/express/lib/router/layer.js
Layer.prototype.handle_request = function handle(req, res, next) {
  // 获取此层 layer 的处理函数
  // 每层 layer 对应不同的处理函数，保存到路由数组 this.stack 中
  var fn = this.handle;

  try {
    // 执行具体的处理函数
    fn(req, res, next);
  } catch (err) {
    next(err);
  }
};

```

- fn(req, res, next) 相当于执行 app.js 启动时绑定的处理函数，以 app.get('/') 绑定的处理函数为例子

```js

// app.js

// 添加路径处理函数
app.get('/', function(req, res){
  // 将会通过 Layer.prototype.handle_request 的 fn(req, res, next) 的函数执行到这里
  console.log('handle 1')
  // 返回结果
  res.send('Hello World!')
  console.log('handle 2')
})

```



## 三. express 模拟实现



#### 1. 核心文件 express.js

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
      // 获取其中一个路由处理方法
      let route = app.routes[index];
      // 下一个路由处理方法的下标
      index++;
      // 如果是中间件
      if (route.method == "middleware"){
        // 如果发生了错误，则找错误处理函数
        if(err){
          // 错误处理函数有四个参数
          if (route.handler.length == 4) {
            // console.log("错误处理啦", req, res);
            route.handler(err, req, res, next);
          } else { 
            // 没有找到，继续向下找错误处理函数 
            next(err);
          }
        }else{
          // 正常中间件处理
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
            // 没有找到，继续遍历下一个处理函数
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

  // 定义路由数组，存放处理函数，包括中间件和真实路由
  app.routes = []; 

  // 获取所有请求方法，比如常见的 GET/POST/DELETE/PUT ...
  let methods = http.METHODS; 
  methods.forEach(method => {
    // 变成小写，这样就可以通过 app.get(), app.post() 调用
    method = method.toLocaleLowerCase()
    app[method] = function (path, handler) {
      // 第一个是路径参数，第二是处理函数
      // 比如 app.get('/hello',function(){ console.log('hello') })
      // 参数 laryer = { 'method' : 'get', 'path' : '/hello', 'handler' : function(){ console.log('hello')}
      let layer = {
        method,
        path,
        handler,
      }
      // layer 理解为层，请求将会经过一层层的处理，好像剥洋葱一样
      // 将处理对象保存到路由数组中
      app.routes.push(layer)
    }
  })

  // 添加中间件的函数，跟真实路由类似，只是通过来 method 区分
  app.use = function(path, handler) {
    app.routes.push({
      method:"middleware",
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


#### 2. 测试文件 index.js

```js
const express = require('./express.js');
const app = express();

// 添加中间件，所有路径都会执行
app.use('/',function (req,res,next) {
  console.log('middleware for all req ',req.url);
  next();
});

// 只对 test 路径执行的中间件
app.use('/test', function (req,res,next) {
  console.log('middleware only for test',req.url);
  next();
});

// get 请求
app.get("/hello",function(req,res){
  res.end("hello");
});
// post 请求
app.post("/world",function(req,res) {
  res.end("world");
});

// 如果以上路由都匹配不了，还有兜底路由
app.all("all",function(req,res) {
  console.log('all for 404',req.url);
  res.end("all for 404")
});

// 处理错误的中间件，用四个参数来区分
app.use('/',function (err, req, res, next) {
  console.log('get error msg',err);
  res.end(err);
});


// 监听端口
app.listen(8080,function() {
  // body...
  console.log('run express')
});

```