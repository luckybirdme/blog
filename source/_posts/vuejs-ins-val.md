---
title: vuejs 组件通信方式
date: 2019-12-02 11:04:26
tags: vuejs
---

> this.$parent 指向父组件实例，可调用父实例的属性和方法，而且可以层层递进方式父实例，比如 this.$parent.$parent 。
> this.$children 以数组方式存放了子组件实例，无法保证子组件的顺序，访问方式 this.$childrend[0] 。
> this.$refs 以对象方式存放了子组件实例，需要先给每个子组件标记属性 ref='usernameInput' 。
> $emit 和 $on 用于触发和侦听 Vue 实例事件，即事件总线机制 EventBus。
> $attrs 和 $listener 用于爷孙之间的组件通信，采用透传方式实现。
> provide/inject, 父组件提供变量，注入到子组件中，主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中。

<!-- more -->


## 一. $parent, 子组件访问父组件。
#### 1. 子组件只有一个父组件实例
```js
this.$parent.msg
```
#### 2. 通过层层递进方式访问
```js
this.$parent.$parent.msg
```


## 二. $children, 父组件访问子组件。

#### 1. 通过数组方式访问
```js
// 输出所有子组件的msg数据
for(let i=0;i<this.$children.length;i++){
   console.log(this.$children[i].msg);
}
```
#### 2. 当前实例的直接子组件。需要注意 $children 并不保证顺序，也不是响应式的


## 三. $refs, 父组件访问子组件。
#### 1. 在父组件通过 ref 标记子组件
```html
<base-input ref="usernameInput"></base-input>
```

#### 2.在父组件中，调用子组件的方法
```js
methods: {
  // 用来从父级组件聚焦输入框
  focus: function () {
    this.$refs.usernameInput.focus()
  }
}
```

#### 3. $refs 只会在组件渲染完成之后生效，并且它们不是响应式的。这仅作为一个用于直接操作子组件的“逃生舱”——你应该避免在模板或计算属性中访问 $refs。



## 四. $emit 和 $on, 触发和侦听 Vue 实例事件。
#### 在不同组件中引入事件总线 EventBus, 达到组件通信的作用。
#### 1. 编写 EventBus.js 文件
```js
import Vue from 'vue'
export const EventBus = new Vue()
```

#### 2. 在组件 A 中触发 EventBus 事件
```html
<template>
    <button @click="sendMsg()">-</button>
</template>
 
<script> 
import { EventBus } from "../EventBus.js";
export default {
  methods: {
    sendMsg() {
      EventBus.$emit("showMsg", '来自 A 页面的消息');
    }
  }
}; 
</script>
```

#### 3. 在组件 B 中侦听 EventBus 事件
```html
<template>
    <div>{{msg}}</div>
</template>
<script> 
import { EventBus } from "../EventBus.js";
export default {
  data(){
    return {
      msg: ''
    }
  },
  mounted() {
    EventBus.$on("showMsg", (msg) => {
      // A发送来的消息
      this.msg = msg;
    });
  }
};

</script>
```


## 五. $attr 和 $listener, 子孙组件访问爷爷组件
#### $attr, 包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。
#### $listener, 包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。

#### 1. 爷爷组件
```html
<template>
    <div id="app">
        <Father v-bind:info="info" v-on:getData="getData"></Father>
    </div>
</template>
 
<script>
    import Father from "./father";
    export default {
      name: 'Grandfather',
      components: {Father},
      data() {
          return {
            info:'我是来自爷爷组件',
          }
      },
      methods: {
        getData (val) {
           this.msg = val
           console.log(val)
        }
      }
    }
</script>
```

#### 2. 父组件

```html
<template>
    <div>
    	<!-- 透传爷爷组件的属性 -->
      <Son v-bind="$attrs" v-on="$listener"></Son>
    </div>
</template>
<script>
    import Son from "./son";
    export default {
      name: 'Father',
      components: {Son}
    }
</script>
```

#### 3. 子孙组件
```html
<template>
    <div>
    	<!-- 获取爷爷组件的属性 -->
      <div >{{this.$attrs.info}}</div>
    </div>
</template>
<script>
    export default {
      name: 'Son',
      mounted() {
      	// 触发爷爷组件事件
        this.$emit('getData','我来自孙子组件')
      }
    }
</script>
```


## 六. provide 和 inject , 不同层次组件之间通信
#### 1. 以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。
#### 2. 绑定并不是可响应的。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的属性还是可响应的。
#### 3. 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中。

```js
// 父级组件提供 'foo'
var Provider = {
  provide: {
    foo: 'bar'
  },
  // ...
}

// 子组件注入 'foo'
var Child = {
  inject: ['foo'],
  created () {
    console.log(this.foo) // => "bar"
  }
  // ...
}
```
