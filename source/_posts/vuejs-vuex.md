---
title: vuejs 状态管理 vuex
date: 2019-12-03 08:30:45
tags: vuejs
---

> Vuejs 组件关系包括父子组件，兄弟(同级)组件，跨级组件等，为实现复杂业务场景，组件间不可避免需要进行通信。
> 通信方法包括 props, $refs, $parent, $children, provide/inject, $attrs/$listener, eventBus 。
> Vuejs 提供 Vuex 状态管理，集中存储和管理组件状态，以达到组件通信的作用。
> Vuex 实质是借助 Vue 实例的 data 属性，保存数据，达到响应式的效果。

<!-- more -->

## 一. Vuex 原理
#### 1. 系统架构
![](http://qiniucdn.luckybird.me/blog/img/2019/vuex.png)

#### 2. 原理分析
- 借助 Vue 实例的 data 属性，保存 Vuex 的 state 数据
- 可通过 store.state.key, 直接获取 state 值。
- 也可通过 getters.getKeyVal, 获取经过计算加工的 state 值，类似 Vue 实例的 computed 属性。
- 同步修改 state 值，利用 mutations 定义的方法来触发修改。
- 异步修改 state 值，利用 actions 定义的方法来触发修改。
- 如果为了区分不同 store, 可利用 modules 来实现命名区分。

![](http://qiniucdn.luckybird.me/blog/img/2019/vue-vuex.png)



## 二. 使用 Vuex
#### 1. 引入 Vuex 插件
```js
import Vuex from 'vuex'
Vue.use(Vuex)
```

#### 2. 定义 store，并挂到根实例
```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
const app = new Vue({
  el: '#app',
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store
})

```

#### 3. 获取 state 值
```js
// 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}

```

#### 4. 修改 state 值
```js
this.$store.commit('increment')

```



## 三. Vuex 核心
#### 1. state, 保存数据，并可直接获取。
- Vuex 使用单一状态树——是的，用一个对象就包含了全部的应用层级状态。
- 存储在 Vuex 中的数据和 Vue 实例中的 data 遵循相同的规则，例如状态对象必须是纯粹 (plain) 的。

```js
// 定义
const store = new Vuex.Store({
  state: {
    count: 0
  }
})

// 获取
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}

// 辅助函数 mapState 
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'
export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}

// 或者
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```


#### 2. getters, 计算属性，类似 Vue 的 computed 属性。
```js
// 定义 store
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  // 对 todos 的数据进行过滤处理
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})

// 通过 getters 方法获取计算属性值
const Todo = {
  template: `<div>Todo</div>`,
  computed: {
    localdoneTodos () {
      return this.$store.getters.doneTodos
    }
  }
}


// 辅助函数 mapGetters 
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}

// 或者
mapGetters({
  // 把 `this.doneCount` 映射为 `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```

#### 3. mutations, 同步触发修改 state 数据，立即生效。

```js
// 定义 store
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})

// 触发修改
this.$store.commit('increment')


// 如果要传递参数
mutations: {
  // 接收参数
  increment (state, payload) {
    state.count += payload.amount
  }
}

// 传递参数，并触发修改
this.$store.commit('increment',{ amount : 10})



// 辅助函数 mapMutations 
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}

```



#### 4. actions, 异步触发 state 修改，需要等待异步回调。
```js
// 定义 store
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
  	// 支持同步修改 state 
  	// 函数接受一个与 store 实例具有相同方法和属性的 context 对象
  	// 因此你可以调用 context.commit 提交一个 mutation，或者调用 context.state 和 context.getters 
  	// 但是此 context 对象为什么不是 store 实例本身了。
  	increment (context) {
      context.commit('increment')
    },
  	// 异步触发 state 修改
	  incrementAsync ({ commit }) {
	    setTimeout(() => {
	      commit('increment')
	    }, 1000)
	  }
  }
})


// 异步触发修改 state

// 以载荷形式分发
this.$store.dispatch('incrementAsync')


// 辅助函数 mapActions
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

      // `mapActions` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```


#### 5. modules, 模块化管理 store

```js

// 定义 store 模块
const moduleA = {
	namespaced: true,
  state: {
  	count:0
  },
  mutations: {
  	increment (state) {
      state.count++
    }
  }
}
const moduleB = {
	namespaced: true,
  state: {
  	count:0
  },
  mutations: {
  	increment (state) {
      state.count++
    }
  }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

// -> moduleA 的状态
this.$store.state.a 
// 触发 moduleA 修改 state
this.$store.commit('a/increment');

// -> moduleB 的状态
this.$store.state.b 
// 触发 moduleB 修改 state
this.$store.commit('b/increment');


// 模块化下 getters 和 actions 方式的参数
modules: {
  foo: {
    namespaced: true,

    getters: {
      // 在这个模块的 getter 中，`getters` 被局部化了
      // 你可以使用 getter 的第四个参数来调用 `rootGetters`
      someGetter (state, getters, rootState, rootGetters) {
        getters.someOtherGetter // -> 'foo/someOtherGetter'
        rootGetters.someOtherGetter // -> 'someOtherGetter'
      },
      someOtherGetter: state => { ... }
    },

    actions: {
      // 在这个模块中， dispatch 和 commit 也被局部化了
      // 他们可以接受 `root` 属性以访问根 dispatch 或 commit
      someAction ({ dispatch, commit, getters, rootGetters }) {
        getters.someGetter // -> 'foo/someGetter'
        rootGetters.someGetter // -> 'someGetter'

        // 通过  dispatch 和 commit 第三个参数  { root: true } 确定触发具体模块中的方法 
        dispatch('someOtherAction') // -> 'foo/someOtherAction'
        dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

        commit('someMutation') // -> 'foo/someMutation'
        commit('someMutation', null, { root: true }) // -> 'someMutation'
      },
      someOtherAction (ctx, payload) { ... }
    }
  }
}

```