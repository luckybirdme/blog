---
title: vuejs 组件通信方式
date: 2019-12-02 10:47:17
tags: vuejs
---

> 父组件向子组件传递
> 子组件向父组件传递
> 爷组件向孙组件传递
> 孙组件向爷组件传递
> 同级组件向组件传递

<!-- more -->

组件通信方向|使用通信方法
-|-
父->子| props, vm.$refs, vm.$children, provide / inject, eventBus, Vuex
子->父| vm.$on / vm.$emit, vm.$parent, eventBus, Vuex 
爷->孙| vm.$attrs, provide / inject, eventBus, Vuex
孙->爷| vm.$listeners, eventBus, Vuex
子->子| eventBus, Vuex



#### props

1. 说明:
子组件接收来自父组件的数据，可以是数组或对象，对象允许配置高级选项，如类型检测、自定义验证和设置默认值。

2. 示例:

```html

<html>
<head>
    <script src="https://cdn.bootcss.com/vue/2.5.20/vue.min.js"></script>
</head>
<body>
<!-- 父组件 -->
<div id="app">
    <!-- 子组件 -->
    <child message="hello world!" blog-title='I am fine'>父组件传递给子组件</child>
</div>

<script>
    // 子组件
    Vue.component('child', {
      // 声明 props, 接收父组件变量
      // 横岗命名和驼峰命名会自动转换
      props: ['message','blogTitle'],
      // 直接在 template 中使用
      template: '<h1>{{ message }}</h1>',
      // 同样也可以在 vm 实例中像 this.message 这样使用
      created: function(){
        console.log('message',this.message);
        console.log('blogTitle',this.blogTitle);
      }
    })
    // 创建根实例
    new Vue({
      el: '#app'
    })
</script>
</body>
</html>

```


#### vm.$refs

1. 说明:
保存了一个对象，该对象是注册过 ref attribute 的所有 DOM 元素和组件实例。父组件可以通过给子组件注册 ref 属性，达到访问子组件实例的效果。

2. 示例:

```html

<html>
<head>
    <script src="https://cdn.bootcss.com/vue/2.5.20/vue.min.js"></script>
</head>
<body>
<!-- 根组件 -->
<div id="app">
    <!-- 父组件 -->
    <parent>
        <!-- 子组件 -->
    </parent>
</div>

<script>
    // 父组件
    Vue.component('parent', {
      created: function(){
        console.log('parent')
      },
      // 子组件
      // 通过 ref 标记子组件 id
      template: '<child ref="myChild">test</child>',
      mounted: function(){
        // 父组件通过子组件 ref 的值访问子组件实例
        // 访问子组件的 data , 以及 methods 的方法
        console.log(this.$refs.myChild);
        console.log(this.$refs.myChild.childDataVal);
        this.$refs.myChild.sayHi();
      }
    });
    // 子组件
    Vue.component('child', {
      data: function () {
        return {
          childDataVal: 'childDataVal'
        }
      },
      methods: {
        sayHi: function(){
          console.log('I am child');
        }
      },
      template: '<h1>I am child</h1>',
      created: function(){
      }
    })
    // 创建根实例
    new Vue({
      el: '#app'
    })
</script>
</body>
</html>

```




#### vm.$on / vm.$emit

1. 说明

- vm.$on 监听指定组件实例上的自定义事件，事件可以由该实例通过 vm.$emit 触发。请注意，vm.$emit和vm.$on的事件必须在一个公共的组件实例上，才能够触发。
- 一般用于父组件侦听子组件的事件，然后在子组件触发，以便子组件向父组件通信，其中父组件的回调函数可以接收子组件传递的额外参数。
- 如果只想监听一次，可使用 vm.$once, 或者使用 vm.$off 移除监听事件。


2. 示例


```html
<html>
<head>
    <script src="https://cdn.bootcss.com/vue/2.5.20/vue.min.js"></script>
</head>
<body>
<!-- 根组件 -->
<div id="app">
    <!-- 父组件 -->
    <parent>
        <!-- 子组件 -->
    </parent>
</div>

<script>
    // 父组件
    Vue.component('parent', {
      created: function(){
        console.log('parent created')
      },
      methods: {
        getFromChild: function(msg){
          console.log('getFromChild ' + msg);
        }
      },
      // 侦听子组件的 sendToParent 事件
      template: '<child v-on:sendToParent="getFromChild">I am child</child>',
      mounted: function(){
      }
    });
    // 子组件
    Vue.component('child', {
      methods: {
        // 触发 sendToParent 事件
        clickEvent: function(){
          this.$emit('sendToParent','hello');
        }
      },
      // 绑定点击事件
      template: '<button v-on:click="clickEvent">Click me</button>',
      created: function(){
        console.log('child created')
      },
      mounted: function(){
      }
    })
    // 创建根实例
    new Vue({
      el: '#app'
    })
</script>
</body>
</html>

```


#### vm.$attrs 

1. 说明
- props 是父组件传递值到子组件，如果子组件没有全部获取父组件传递的 props 值，那剩余的值会直接添加到子组件的节点上的 DOM 上，当做普通属性。
- props 来自爷组件，可以通过父组件，再传递给孙组件，但是必须在父组件中全部获取 props 的值，然后透传个孙组件，那么会显得很麻烦。
- vm.$attrs 包含了爷组件中未被父组件识别并获取的 props 值，可以在父组件中通过 v-bind="$attrs" 绑定到孙组件，达到透传 props 的作用，也实现了爷组件向孙组件通信的功能。


2. 示例

```html

<html>
<head>
    <script src="https://cdn.bootcss.com/vue/2.5.20/vue.min.js"></script>
</head>
<body>
<!-- 根组件 -->
<div id="app">
  <!-- 爷组件 -->
  <grandfather>
    <!-- 父组件 -->
        <!-- 子组件 -->
  </grandfather>

</div>

<script>

    // 爷组件
    Vue.component('grandfather', {
      created: function(){
        console.log('grandfather created');
      },
      // 传递信息给父组件，子组件
      template: '<parent toparent="toparent hello" tograndson="tograndson hello" >I am parent</parent>',
      mounted: function(){
      }
    });
    // 父组件
    Vue.component('parent', {
      // 接收爷组件的信息
      props: ["toparent"],
      created: function(){
        console.log('parent created');
      },
      // 透传未被识别的 props 值给孙组件
      template: '<child v-bind="$attrs">I am child</child>',
      mounted: function(){
        console.log(this.toparent);
        // 未被识别并获取的爷组件信息 props
        console.log(this.$attrs);
      }
    });
    // 子组件
    Vue.component('child', {
      // 获取爷组件的信息
      props: ["tograndson"],
      methods: {
      },
      // 绑定点击事件
      template: '<h1>{{ tograndson }}</h1>',
      created: function(){
        console.log('child created')
      },
      mounted: function(){
        console.log(this.tograndson);
      }
    })
    // 创建根实例
    new Vue({
      el: '#app'
    })
</script>
</body>
</html>

```


#### vm.$listeners

1. 说明
- vm.$on / vm.$emit 要是是同一个组件实例上侦听和触发事件，比如父组侦听子组件，子组件触发父组件事件。
- 对于爷组件，无法直接侦听子组件的事件，但是父组件可以通过 vm.$listeners 让爷组件侦听到孙组件触发的事件。
- vm.$listeners 包含了父作用域中的 v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件，在创建更高层次的组件时非常有用。

2. 示例

```html

<html>
<head>
    <script src="https://cdn.bootcss.com/vue/2.5.20/vue.min.js"></script>
</head>
<body>
<!-- 根组件 -->
<div id="app">
  <!-- 爷组件 -->
  <grandfather>
    <!-- 父组件 -->
        <!-- 子组件 -->
  </grandfather>

</div>

<script>

    // 爷组件
    Vue.component('grandfather', {
      methods: {
        getMsg: function(msg){
          console.log(msg)
        }
      },
      created: function(){
      },
      // 爷组件侦听孙组件的事件
      template: '<parent v-on:tograndfather="getMsg" >I am parent</parent>',
      mounted: function(){
      }
    });
    // 父组件
    Vue.component('parent', {
      created: function(){
        console.log('parent created');
      },
      // 让爷组件可以侦听孙组件的事件
      template: '<child v-on="$listeners">I am child</child>',
      mounted: function(){
        console.log(this.$listeners);
      }
    });
    // 子组件
    Vue.component('child', {
      methods: {
        // 孙组件触发事件，通过父组件透传给爷组件
        sayHi: function(){
          this.$emit("tograndfather",'from my grandson');
        }
      },
      // 绑定点击事件
      template: '<button v-on:click="sayHi">tell my grandfather</button>',
      created: function(){
        console.log('child created')
      },
      mounted: function(){
      }
    })
    // 创建根实例
    new Vue({
      el: '#app'
    })
</script>
</body>
</html>

```





#### vm.$parent / vm.$children / vm.$root

1. 说明
- vm.$parent 当前组件实例的父组件实例。
- vm.$children 当前组件实例的子组件实例，以数组形式保存多个子组件实例。
- vm.$root 根节点，一般是在最顶层的 el 节点

2. vm.$parent 和 vm.$parent 主要目的是作为访问组件的应急方法，不建议过多事情，更推荐用 props 和 vm.$emit


```html

<html>
<head>
    <script src="https://cdn.bootcss.com/vue/2.5.20/vue.min.js"></script>
</head>
<body>
<!-- 根组件 -->
<div id="app">
    <!-- 父组件 -->
    <parent>
        <!-- 子组件 -->
    </parent>
</div>

<script>
    // 父组件
    Vue.component('parent', {
      created: function(){
        console.log('parent created')
      },
      methods: {
        sayHello: function(){
          console.log('I am parent');
        }
      },
      // 子组件
      template: '<child >I am child</child>',
      mounted: function(){
        // 父组件通过 $children 访问子组件
        console.log(this.$children);
        console.log(this.$children[0].childDataVal);
        this.$children[0].sayHi();
      }
    });
    // 子组件
    Vue.component('child', {
      data: function () {
        return {
          childDataVal: 'childDataVal'
        }
      },
      methods: {
        sayHi: function(){
          console.log('I am child');
        }
      },
      template: '<h1>I am child</h1>',
      created: function(){
        console.log('child created')
      },
      mounted: function(){
        // 子组件通过 $parent 访问子组件
        console.log(this.$parent);
        this.$parent.sayHello();
      }
    })
    // 创建根实例
    new Vue({
      el: '#app'
    })
</script>
</body>
</html>

```


#### provide / inject

1. 说明
这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。

2. provide 和 inject 主要在开发高阶插件/组件库时使用，并不推荐用于普通应用程序代码中。

```js

// 父级组件提供 'foo'
var Provider = {
  provide: {
    foo: 'bar'
  },
}

// 子组件注入 'foo'
var Child = {
  inject: ['foo'],
  created () {
    // => "bar"
    console.log(this.foo) 
  }
}
```


#### eventBus

1. 说明
- 事件总线 eventBus 可以理解是组件之间的沟通桥梁，所有组件共用的事件中心，可以向该中心监听事件和触发事件。
- vm.$on 和 vm.$emit 也是基于类似的机制，但是限定在同一个组件实例中，而 eventBus 则设计成一个共用的全局实例，所以支持在不同组件实例中使用。

2. 示例

```html

<html>
<head>
    <script src="https://cdn.bootcss.com/vue/2.5.20/vue.min.js"></script>
</head>
<body>
<!-- 根组件 -->
<div id="app">
    <!-- 父组件 -->
    <parent>
        <!-- 子组件 -->
    </parent>
</div>

<script>
    // 公共的事件总线
    // 绑定到 Vue 的原型对象中，以便所有组件都能使用 eventBus
    Vue.prototype.$eventBus = new Vue();

    // 父组件
    Vue.component('parent', {
      created: function(){
        console.log('parent created');
      },
      methods: {
      },
      template: '<child >I am child</child>',
      mounted: function(){
        // 侦听 sendToParent 事件，可以在任意组件中触发
        this.$eventBus.$on("sendToParent", function(msg){
          console.log('getFromChild ' + msg);
        });

        // 触发 sendToChild 事件，可以在任意组件接收
        setTimeout( ()=>{
          this.$eventBus.$emit("sendToChild","hi");
        });
      }
    });
    // 子组件
    Vue.component('child', {
      methods: {
        // 触发 sendToParent 事件，可以在任意组件接收
        clickEvent: function(){
          this.$eventBus.$emit('sendToParent','hello');
        }
      },
      // 绑定点击事件
      template: '<button v-on:click="clickEvent">Click me</button>',
      created: function(){
        console.log('child created')
      },
      mounted: function(){
        // 侦听 sendToChild 事件，可以在任意组件中触发
        this.$eventBus.$on("sendToChild", function(msg){
          console.log('getFromParent ' + msg);
        });
      }
    })
    // 创建根实例
    new Vue({
      el: '#app'
    })
</script>
</body>
</html>

```

#### vuex

1. 说明

- Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
- 通俗地将，Vuex 可以集中存储和管理组件的状态，组件之间可以通过这些状态来进行通信，因此 Vuex 合适在不同层级的组件通信。
- Vuex 合适中大型单页应用，可以更好地在组件之间管理状态。而简单的子父组件通信，可以采用上面提到的 props 和 vm.$emit

2. [示例](http://blog.luckybird.me/2019/12/03/vuejs-vuex/)

