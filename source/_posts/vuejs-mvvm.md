---
title: vuejs 数据双向绑定
date: 2019-11-21 08:22:04
tags: vuejs
---

> 数据双向绑定，即视图数据改变，模型数据立即更新，反之亦然。
> 该模式又称为 MVVM，即 Model-View-ViewModel。

<!-- more -->



## 一，数据双向绑定
##### 1. MVVM 模式使得视图 view 和 model 交互变得极为方便， view 的改变立即同步到 model ，model 的改变也可立即同步到 view，数据流如下图所示。
![](/img/2019/MVVM.png)

##### 2. vuejs 实现 MVVM 模式主要通过 Object.defineProperty() 来侦听属性的 set 和 get 的动作，达到数据立即同步，双向绑定的效果。由于 IE8 的 JS 原生对象不支持此方法，也无法通过 Polyfill 来兼容，所以 vuejs 无法兼容 IE8。
![](/img/2019/vuejs_mvvm_two.png)


- Observer，数据监听器，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者，内部采用Object.defineProperty的getter和setter来实现。相当于Model层观察vue实例中的data数据，当数据发生变化时，通知Watcher订阅者。

- Compile，指令解析器，它的作用对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数。解析器位于View层，初始化View的视图，将数据变化与更新函数绑定，传给Watcher订阅者。

- Watcher，订阅者，作为连接 Observer 和 Compile 的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数。是整个模型的核心，对应ViewModel层，连接Observer和Compile。所有的Watchers存于Dep订阅容器的数组中，Watcher将Observer监听到的数据变化对应相应的回调函数，处理数据，反馈给View层更新界面视图。

- Dep，订阅者容器，内部维护了一个数组，用来收集订阅者（Watcher），数据变动触发 notify 函数，再调用订阅者的 update 方法。



##### 3. vuejs 实现 MVVM 方式 是对数据（Model）进行劫持，当数据变动时，数据会触发劫持时绑定的方法，对视图进行更新；而 Angular.js 采取的是另外脏检查机制，当发生了某种事件（例如输入），会检查新的数据结构和之前的数据结构是否发生来变动，来决定是否更新视图。


## 二，原生 JS 实现 MVVM 模式

```html
<html>
	<head>
	</head>
	<body>
		<div id="app">
			<input type="text" v-model="message" />
			<span>{{ message }}</span>
		</div>
	<script>

		// Watcher, 属性变化时，会跟着变化
		function Watcher (vm, node, name, nodeType) {
			this.name = name;
			this.node = node;
			this.vm = vm;
			this.nodeType = nodeType;
			this.value = '';
			// 初始化时，设置默认值
			this.update();
		}
		Watcher.prototype = {
			// set 触发时，进行更新
			update: function () {
				// 获取更新值
				console.log(this.nodeType,'get',this.value);
				if(this.value == this.vm[this.name]){
					console.log(this.nodeType,'same',this.value);
					return;
				}
				this.value = this.vm[this.name]; 
				console.log(this.nodeType,'update',this.value);

				// 显示 text
				if (this.nodeType == 'text') {
					this.node.nodeValue = this.value;
				}
				// 显示 input
				if (this.nodeType == 'input') {
					this.node.value = this.value;
				}
			}
		}

		// 用于存放所有的 Watcher
		function Dep() {
			this.subs = [];
		}
		Dep.prototype = {
			// 添加 Watcher
			addSub: function(sub) {
				this.subs.push(sub);
			},
			notify: function() {
				// 遍历更新 Watcher
				this.subs.forEach(function(sub) {
					sub.update();
				});
			}
		};
		
		// Observer, 用于观察属性变化，即绑定属性的 set 和 get
		// 当 set 触发时，通知所有 Watcher 更新
		function Observer(data,vm) {
			if (!data || typeof data !== 'object') {
				return;
			}

			// 容器
			this.dep = new Dep();
			// 取出所有属性遍历，绑定 set 和 get 
			for(var key in data){
				defineReactive(vm, key, data[key], this.dep);
			}
		};
		// 通过 Object.defineProperty 进行 set 和 get 绑定
		// 此方法是 ES5 才有，所以无法兼容 IE8 
		function defineReactive(vm, key, val, dep) {
			Object.defineProperty(vm, key, {
				enumerable: true, // 可枚举
				configurable: false, // 不能再define
				get: function() {
					// 获取 val 时被触发
					console.log(key,'get');
					return val;
				},
				set: function(newVal) {
					// 设置 val 时被触发
					var locVal = val;
					console.log(key,'set,locVal:' + locVal);
					if(locVal == newVal){
						console.log(key,'set,same:' + locVal);
						return;
					}
					console.log(key,'set,newVal:' + newVal);
					val = newVal;
					// 通知所有 Watcher 更新，实现数据双向绑定
					// model 同步到 view
					dep.notify();
				}
			});
		}

		// 递归解析 node 节点
		function nodeToFragment(node,vm,dep){

			if (node.hasChildNodes()) {
				var children = node.childNodes;
				for (var i = 0; i < children.length; i++) {
					nodeToFragment(children[i],vm,dep);				
				}
			}else{
				compile(node,vm,dep);
			}
		}
		
		// 翻译事件命令和变量
		function compile (node, vm, dep) {
			// 节点类型为元素
			if (node.nodeType === 1) {
				var attr = node.attributes;
				// 解析属性
				for (var i = 0; i < attr.length; i++) {
					// 翻译 v-model 命令
					if (attr[i].nodeName == 'v-model') {
						var name = attr[i].nodeValue; 
						// 侦听 input 输入
						node.addEventListener('input', function (e) {
							// 将 input 变化会同步到 data
							// view 同步到 model
							// 触发属性的 set 方法
							console.log('input','listen,set')
							vm[name] = e.target.value;
						});
						// 首次运行会触发
						console.log('input','init and get');
						 // 将默认 data 的值赋给该 input
						node.value = vm[name];
						node.removeAttribute('v-model');

						// 添加 Watcher，当 data 变化将会自动同步到 input 
						var w = new Watcher(vm, node, name, 'input');
						dep.addSub(w);
					}
				};
				
			}
			// 节点类型为text
			if (node.nodeType === 3) {
				// 匹配变量{{}}
				var reg = /\{\{(.*)\}\}/;
				var arr = reg.exec(node.nodeValue);
				if(arr !== null) {
					// 获取匹配到的字符串
					var name = arr[1].trim();
					// 添加 Watcher, 当 data 变化将会自动同步到 text 
					var w = new Watcher(vm, node, name, 'text');
					dep.addSub(w);
				
				}
			}
		}
	
		// Vue 对象，实现 MVVM
		function Vue (options) {
			this.data = options.data;
			var data = this.data;
			// 绑定 data 属性的 set 和 get 
			var ob = new Observer(data, this);
			var id = options.el;
			var app  = document.querySelector(id);
			// 进行文档解析，并添加 Watcher
			var dom = nodeToFragment(app, this, ob.dep);
		}
		var vm = new Vue({
			el: '#app',
			data: {
				message: 'hello world'
			}
		});
	
	</script>
	</body>
</html>
```