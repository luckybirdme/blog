---
title: vuejs 生命周期
date: 2019-11-23 16:06:24
tags: vuejs
---

> vuejs 生命周期主要包括 beforeCreate,created,beforeMount,mounted,beforeUpdate,updated,beforeDestory,destoryed

<!-- more -->

## 一，vuejs 生命周期

#### 1. 流程图
![](http://qiniucdn.luckybird.me/blog/img/2019/lifecycle.png)

#### 2. 代码解析
```html
<!DOCTYPE html>
<html>
<head>
    <script src="../lib/vue-2.6.10.js"></script>
</head>
<body>
    <div id="app">
        {{ msg }}
        <input v-model="val">
    </div>
    <div>
        <button type="button" onclick="destory()">destory</button>
    </div>
    <script type="text/javascript">
        var app = new Vue({
            el: '#app',
            data: {
                msg: 'Hello Vue!',
                val: ''
            },
            // 在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用。
            beforeCreate: function() {
                window.console.log("beforeCreate")
                // undefined
                window.console.log(this.msg)
                // undefined
                window.console.log(this.$el)
            },
            // 实例已完成数据观测 (data observer)，属性和方法的运算，watch/event 事件回调。
            // 然而，挂载阶段还没开始，$el 属性目前不可见。
            created: function() {
                window.console.log("created")
                // had value
                window.console.log(this.msg)
                // undefined
                window.console.log(this.$el)
            },
            // 在挂载开始之前被调用：相关的 render 函数首次被调用。
            beforeMount: function() {
                window.console.log("beforeMount")
                // had value
                window.console.log(this.msg)
                // had virtual dom
                window.console.log(this.$el)
            },
            // el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子。
            mounted: function() {
                window.console.log("mounted")
                // had value
                window.console.log(this.msg)
                // replace el by virtual dom
                window.console.log(this.$el)
            },
            // 数据更新前调用，发生在虚拟 DOM 打补丁之前。
            // 这里适合在更新之前访问现有的 DOM，比如手动移除已添加的事件监听器。
            beforeUpdate: function() {
                window.console.log("beforeUpdate")
                window.console.log(this.msg)
                window.console.log(this.$el)
            },
            // 数据更新完毕后执行，或者由于数据更改导致的虚拟 DOM 重新渲染和打补丁。
            updated: function() {
                window.console.log("updated")
                window.console.log(this.msg)
                window.console.log(this.$el)
            },
            // 实例销毁之前调用。在这一步，实例仍然完全可用。
            beforeDestory: function() {
                window.console.log("beforeDestory")
                window.console.log(this.msg)
                window.console.log(this.$el)
            },
            // Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。
            destoryed: function() {
                window.console.log("destoryed")
                window.console.log(this.msg)
                window.console.log(this.$el)
            }
        });
    </script>
</body>
</html>
```


## 二，注意事项
#### 1. mounted 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都挂载完毕，可以用 vm.$nextTick 。 updated 也有同样的特点。
```js
mounted: function () {
  this.$nextTick(function () {
    // Code that will run only after the entire view has been rendered
  })
}
updated: function () {
  this.$nextTick(function () {
    // Code that will run only after the entire view has been re-rendered
  })
}
```