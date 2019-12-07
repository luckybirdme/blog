---
title: vuejs 组件通信 props
date: 2019-12-02 10:47:17
tags: vuejs
---

> props 是单向数据流的通信方式，父组件向子组件传递。
> 子组件通过 props 可接收来自父组件的属性，包括静态和动态值。
> 在传递数字、布尔值、数组、对象时，为了避免当成字符串，必须使用 v-bind。
> props 接收变量时，支持数据类型的校验。

<!-- more -->

## 一. 静态传值
#### 1. 父组件传递静态值
```html
<blog-post post-title="hello!"></blog-post>
```

#### 2. 子组件接收值
```js
Vue.component('blog-post', {
  // camelCase in JavaScript
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})

```

#### 3. 另外一种组件方式
```html
<template>
 <p>{{postTitle}}</p>
</template>
<script>
 export default{
  name: 'blog-post',
  props:['postTitle']
 }
</script>
```
注意 HTML 大小写不敏感，所以 camelCase (驼峰命名法) 的 prop 名需要使用其等价的 kebab-case (短横线分隔命名) 命名


## 二. 动态传值

#### 1. 父组件传递动态值
```html
<blog-post v-bind:title="post.title"></blog-post>

```

#### 2. 父组件传递数字，布尔值，数组，对象时，为了避免当成字符串，必须使用 v-bind
```html
<!-- 即便 `42` 是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:likes="42"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:likes="post.likes"></blog-post>

<!-- 即便数组是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>

<!-- 传递整个对象 -->
<blog-post v-bind="post"></blog-post>

```


## 三. 支持数据类型校验
```js
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```



## 四. 单项数据流

#### 1. 在 JavaScript 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变这个对象或数组本身将会影响到父组件的状态。

#### 2. 所有的 prop 都使得其父子 prop 之间形成了一个单向下行绑定：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。

#### 3. 额外的，每次父级组件发生更新时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器的控制台中发出警告。

