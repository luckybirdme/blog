---
title: vuejs 选项参数解析
date: 2019-11-23 17:47:09
tags: vuejs
---
> vuejs 选项主要参数包括: el,data,methods,computed,watch。
> 模块化引用 vuejs 时，还有包括 name,components,props 参数。

<!-- more -->

## 一. el
虚拟 dom 挂载所在的节点
```js
var vm = new Vue({
  el: '#app'
})
```

## 二. data
两种格式
- 使用 script 直接引入，data 是一个纯粹的 key/val 对象。
- 使用 import/require 使用一个模块系统，比如通过 Babel 和 webpack 使用 ES2015 模块，在SFC(Single File Components)下，data 是一个函数，并返回一个纯粹对象。因为组件可能会复用，因此 data 选项必须是一个函数，即闭包函数，以便返回一个 data  对象的拷贝，让每个组件实例各自维护一份的 data 的数据。

```js
// 直接通过 script 引入
var vm = new Vue({
  el: '#app',
  data: {
    key: 'val'
  }
});


// 模块化使用
var Component = Vue.extend({
  data: function () {
    return { 
      a: 1 
    }
  }
})

export default {
  data: function(){
    return {
      key: 'val'
    }
  }
}
```


## 三. name
模块化的组件名称，用于标记和引用组件

```js
// 组件自身定义 name 属性
export default {
  name: 'TestComponent'
}
```

## 四. components

存放引用的组件

```js

// 直接通过 script 引入
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
var vm = new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
});


// 模块化引用
import TestComponent from './components/TestComponent.vue'
export default {
  name: 'app',
  // 引用的组件
  components: {
    TestComponent
  }
}

```

## 五. props 
props 可以是数组或对象，用于接收来自父组件的数据。props 可以是简单的数组，或者使用对象作为替代，对象允许配置高级选项，如类型检测、自定义验证和设置默认值。

```js

export default {
  name: 'TestComponent',
  props: {
    // 检测类型
    height: Number,
    age: {
      // 校验类型
      type: Number,
      // 默认值
      default: 0,
      // 必须传递
      required: true,
      // 校验取值
      validator: function (value) {
        return value >= 0
      }
    }
  }
}
```


## 六. methods

主要存放人为触发性的操作方法，比如按钮点击响应函数。可直接通过 VM 实例访问这些方法，方法中的 this 自动绑定为 Vue 实例，所以不能使用箭头函数来定义，因为箭头函数默认绑定父级上下文。

```js

export default {
  name: 'TestComponent',
  methods: {
    clickButton: function(){
    },
    selectOption () {
    }
  }
}
```


## 七. computed

主要存放响应式属性变化后产生的计算结果，计算属性的结果会被缓存，除非依赖的响应式属性变化才会重新计算。
还有另外一种场景，比如输出内容需要经过复杂的计算，如果直接写在模板里，不方便维护，那么将复杂的计算放到JS里维护，模板里只填写 computed 的变量名即可。

```html

<template>
  <div class="hello">
    <div>
      <input v-model='val'>
      {{ getValLen}}
    </div>
  </div>
</template>

<script>
export default {
  name: 'TestComponent',
  data: function () {
    return {
      val: ''
    }
  },
  computed: {
    // val 改变才会重新计算
    getValLen: function(){
      return this.val.length
    }
  }
}
</script>


<!-- 复杂的计算 -->
<div id="example">
  {{ message.split('').reverse().join('') + number }}
</div>


<!-- 只填写 computed 变量，具体由 JS 完成计算 -->
<template>
  <div>
    <p>{{ reversedMessage }}</p>
  </div>
</template>
 
<script>
export default {
  name: 'test1',
  data () {
    return {
      message: 'hello world',
      number: 1
    }
  },
  computed: {
    // 字符串反转
    reversedMessage () {
      return this.message.split('').reverse().join('') + this.number
    }
  }
}
</script>
```

## 八. watch

存放一个监听对象，键是需要监听的属性表达式，值是对应回调函数。值也可以是 methods 中的方法名，或者包含选项的对象。Vue 实例将会在实例化时调用 $watch()，遍历 watch 对象的每一个属性。
具体应用实例，比如侦听用户 input 输入，然后发起后台搜索，因此需要 watch 变量实时变化

```html
<script>

var vm = new Vue({
  data: {
    a: 1,
    b: 2,
    c: 3,
    d: 4,
    e: {
      f: {
        g: 5
      }
    }
  },
  methods: {
    show: function(val, oldVal){
        console.log('show',val,oldVal);
    }
  }
  watch: {
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
    // methods 中方法名
    b: 'show',
    // 该回调会在任何被侦听的对象的 property 改变时被调用，不论其被嵌套多深
    c: {
      handler: function (val, oldVal) { /* ... */ },
      deep: true
    },
    // 该回调将会在侦听开始之后被立即调用
    d: {
      handler: 'someMethod',
      immediate: true
    },
    e: [
      'handle1',
      function handle2 (val, oldVal) { /* ... */ },
      {
        handler: function handle3 (val, oldVal) { /* ... */ },
        /* ... */
      }
    ],
    // watch vm.e.f's value: {g: 5}
    'e.f': function (val, oldVal) { /* ... */ }
  }
})
vm.a = 2 // => new: 2, old: 1
</script>


<!-- watch keyword 变化，发起后台搜索 -->
<template>
  <div>
    <p><input v-model="keyword" type="input"></p>
  </div>
</template>
 
<script>
export default {
  name: 'test1',
  data () {
    return {
      keyword:""
    }
  },
  watch: {
    keyword: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
      // search

    }
  }
}
</script>
```
