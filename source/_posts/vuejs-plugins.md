---
title: vuejs 插件
date: 2019-12-02 08:23:18
tags: vuejs
---

> 插件通常用来为 Vue 添加全局功能。
> 常用插件包括 vue-router, vuex 等

<!-- more -->


## 一，插件功能：

#### 1. 添加 Vue 函数的全局方法或者属性。如: vue-custom-element
```js
  //  添加 Vue 对象全局方法或属性
  Vue.myGlobalMethod = function () {
    // 逻辑...
  }
  // 使用
  // Vue.myGlobalMethod()
```

#### 2. 添加 Vue 实例方法，在 Vue.prototype 上实现。
```js
  //  添加 Vue 实例全局方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 逻辑...
  }
  // 使用
  // this.$myMethod()
```

#### 3. 添加全局资源：指令/过滤器/过渡等。如 vue-touch
```js
  // 自定义指令
  Vue.directive('touch', {
    // 元素绑定时
    bind: function () {},
    // 元素解绑时
    unbind: function () {}
  })
  // 使用
  // <a v-touch:tap="onTap">Tap me!</a>

  // 过滤器
  Vue.filter('capitalize', function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1) 
  })

  // 使用
  //{{ message | capitalize }}
  //<div v-bind:id="rawId | formatId"></div>

```

#### 4. 通过全局混入来添加一些组件选项。如 vue-router
```js

  //  注入组件选项
  Vue.mixin({
    // beforeCreate 前执行
    beforeCreate: function () {},
    // 组件卸载前执行
    destroyed: function () {},
    // 组件全局方法
    methods: function () {},
  })

```



## 二，插件开发

#### Vue.js 的插件应该暴露一个 install 方法。这个方法的第一个参数是 Vue 构造器，第二个参数是一个可选的选项对象：

```js

MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function () {
    // 逻辑...
  }

  // 2. 添加全局资源
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
  })

  // 3. 注入组件选项
  Vue.mixin({
    created: function () {
      // 逻辑...
    }
  })

  // 4. 添加实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 逻辑...
  }
}

```



## 三，插件使用

#### 1. 通过全局方法 Vue.use() 使用插件。它需要在你调用 new Vue() 启动应用之前完成：
#### 2. Vue.use 会自动阻止多次注册相同插件，届时即使多次调用也只会注册一次该插件。

```js

// 调用 `MyPlugin.install(Vue)`
Vue.use(MyPlugin)
// 传递参数
Vue.use(MyPlugin, { someOption: true })
new Vue({
  // ...组件选项
})

// 实际例子
// 用 Browserify 或 webpack 提供的 CommonJS 模块环境时
var Vue = require('vue')
var VueRouter = require('vue-router')
// 不要忘了调用此方法
Vue.use(VueRouter)

```