---
title: vuejs 响应式渲染
date: 2020-05-05 22:27:42
tags: vuejs
---

> 数组通过下标修改某个值，vuejs 响应式不会触发渲染
> 对象新增或者删除属性，也不会触发渲染

<!-- more -->


## 一. 对象
#### 1. 由于 Vue 会在初始化实例时对 property 执行 getter/setter 侦听，所以 property 必须在 data 对象上存在才能让 Vue 将它转换为响应式的。Vue 无法检测 property 的添加或移除。


#### 2. Vue.set( target, propertyName/index, value )
Vue.set 向响应式对象中添加一个 property，并确保这个新 property 同样是响应式的，且触发视图更新。此方法支持对象和数组，vm.$set 是此方法的别名

#### 3. 示例

```js

Vue.set(vm.someObject, 'b', 2)
// 或者
this.$set(this.someObject,'b',2)

```


## 二. 数组

#### 1. Vue 将被侦听的数组的变更方法进行了包裹，所以它们也将会触发视图更新。这些被包裹过的方法包括：

```js
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```


#### 2. 变更方法，顾名思义，会变更调用了这些方法的原始数组。相比之下，也有非变更方法，例如 filter()、concat() 和 slice()。它们不会变更原始数组，而总是返回一个新数组。当使用非变更方法时，可以用新数组替换旧数组，即可触发视图渲染。

```js

example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})

```


#### 3. Vue 不能检测以下数组的变动：
- 当你利用索引直接设置一个数组项时，例如：vm.items[indexOfItem] = newValue
- 当你修改数组的长度时，例如：vm.items.length = newLength


可以使用被包裹的变更方法

```js

// 删除元素，并替换新值
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```

或者使用上面对象变更的方法

```js
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// 或者别名调用
vm.$set(vm.items, indexOfItem, newValue)

```

## 三. 强制渲染

上面是针对数组和对象变化无法渲染视图而提出的解决方法，但实际还存在 watch 或 computed 数据变化，依然没有渲染，这时候可考虑采用强制渲染。


#### vm.$forceUpdate()
迫使 Vue 实例重新渲染。注意它仅仅影响实例本身和插入插槽内容的子组件，而不是所有子组件。

```js

this.$forceUpdate(); 

```


