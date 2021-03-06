---
title: vuejs 路由 router
date: 2019-11-25 07:40:31
tags: vuejs
---

> vue-router

<!-- more -->

vue-router 实现单页面路由跳转，提供了三种方式：hash 模式、history 模式、abstract 模式，根据 mode 参数来决定采用哪一种方式，默认是 hash
```js
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

#### 1. hash : 兼容性最好的方法，也是默认模式。
- hash 是 URL 的锚点，比如 http://www.example.com/page/#/id 。 id 代表的是网页中的一个位置，单单改变 # 后的 /#/id，浏览器页面只会跳到相应位置的内容，但不会重新加载网页。
- 即使浏览器刷新，URL 的 # 后面的内容也不会传递到服务器。也就是说 # 是用来指导浏览器动作的，对服务器端完全无用，跟 HTTP 请求无关联。
- 每一次改变 # 后的 URL，都会在浏览器的访问历史中增加一个记录，使用“后退”按钮，就可以回到上一个位置。
- hash 变化会触发 hashchange 事件，通过侦听锚点值的改变，根据不同的值，渲染不同的页面。

```js
// 修改浏览器 url
window.location.hash = '#/id';

// 或者直接替换
window.location.replace('http://www.example.com/page/#/id');

// 监听 hash 变化
window.addEventListener("hashchange", funcRef)

```


#### 2. history : 借助 HTML5 History API 和服务器配置，更友好的 URL。
- HTML5 History API 提供了 pushState 和 replaceState 的方法修改 URL，会产生或修改浏览器的历史记录，继而触发 popstate 事件，但浏览器不会向服务器发起新页面请求。
- history 的 URL 不需要带 # ，比如 http://www.example.com/page/id ， 显得更友好。如果用户刷新浏览器，会把 URL 所有内容传达到服务器，服务器必须匹配所有前端路由到默认首页地址，否则会出现 404。
- 由于服务器屏蔽了 404，所以前端需要单独配置 404 页面。


##### JS 底层实现原理
```js
// 修改浏览器 url
window.history.pushState(stateObject,headTitle,'http://www.example.com/page/id');

// 或者直接替换
window.history.replaceState(stateObject,headTitle,'http://www.example.com/page/id');

// 监听 popstate 变化
window.addEventListener("popstate", funcRef)
```

##### 前端路由 404 页面配置放到最后，因为优先级缘故，此配置不会影响其他精确路由，只起到路由兜底作用。 
```js
const router = new VueRouter({
  mode: 'history',
  routes: [
    {
      // 精确匹配，放在前面，优先匹配
      path: '/user/:id',
      component: UserComponent
    },
    { 
      // 模糊匹配所有路径，放到最后，作为兜底
      path: '*', 
      component: NotFoundComponent 
    }
  ]
})

```

##### 后端服务器配置，以 NGINX 为例子，匹配所有路径到首页地址
```sh
location / {
  try_files $uri $uri/ /index.html;
}
```


#### hash 和 history 模式的区别
- history 模式需要借助 HTML5 History API 新特性，所以得考虑浏览器兼容性。
- history pushState 设置的新URL可以是与当前URL同源的任意URL；而 hash 只可修改#后面的部分。
- history pushState 设置新URL与当前URL一样，也会把记录添加到栈中；而 hash 必须设置不一样的新值才会触发记录添加到栈中。
- history pushState 通过 stateObject 可以添加任意类型的数据记录中，而 hash 只可添加短字符串。
- hash 模式 URL 带有 # ，对搜索引擎优化 SEO 不太好。




#### 3. abstract : 提供一种机制进行浏览历史虚拟管理，非浏览器环境的强制使用此模式，比如 Node.js 服务器端，
##### 如果发现没有浏览器的 API，vue-router 会自动强制进入 abstract 模式，所以 在使用 vue-router 时只要不写 mode 配置即可，默认会在浏览器环境中使用 hash 模式，在移动端原生环境中使用 abstract 模式，具体例子可参考 Weex ，可跨平台来构建 Android、iOS 和 Web 应用。
