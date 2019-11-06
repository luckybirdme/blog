---
title: JS call apply bind 函数
date: 2019-09-26 19:40:59
tags: JavaScript
---
> JavaScript 的函数作用域是定义就决定，但是执行时上文下即 this 会变动；
> 函数运行时会在指定上下文运行，不同 this 指向，会获取到不同对象的属性；
> 为了更改函数运行时的上下文，可通过 call，apply，以及 bind 方式绑定。

<!-- more -->

### 一，call，apply，以及 bind 方式都能绑定指定上下文，区别是
##### 1. 接收参数不一样，apply 第一参与跟 call 和 bind一样，第二个参数是以数组或者类数组 arguments，但是 call 和 bind 第二个参数是一个个地传递的。
```javascript
func.call(self,arg1,arg2,arg3);
func.apply(self,[arg1,arg2,arg3]);
func.bind(self,arg1,arg2,arg3);
```

##### 2. 执行方式不一样，call 和 apply 是马上执行，bind 返回一个待执行函数，后面才手动执行
```javascript
// 马上执行
func.call(self,arg1,arg2,arg3);
func.apply(self,[arg1,arg2,arg3]);
// 只是绑定，返回待执行的函数
var rtn = func.bind(self,arg1,arg2,arg3);
// 现在才执行
rtn();
```

### 二，使用场景
> "Talk is cheap, Show me the code" --- said by Linus Torvalds

- [代码](https://github.com/luckybirdme/luckybirdme.github.io/blob/master/example/js/call-apply-bind.html)

```javascript
			var person =  {
				myName:'Jack',
				getName : function(){
					console.log(this.myName);
				}
			}

			var getName = person.getName;
			// 默认上下文是 window，没有 myName 属性
			getName();
			// 利用 call 函数，传入指定上下文 person ，获得 person 的 myName 属性
			getName.call(person);


			var newPerson =  {
				myName:'Jack',
				hadHouse: 0,
				hadCars: 0,
				getInfo : function(houseNum,carNum){
					this.hadHouse = this.hadHouse + houseNum;
					this.hadCars = this.hadCars + carNum
					console.log(this.myName + ', hadHouse : ' + this.hadHouse + ', hadCars : ' + this.hadCars)
				}
			}

			var getInfo = newPerson.getInfo;
			// 默认上下文是 window，没有 myName 属性
			getInfo();
			// 利用 call 函数，传入指定上下文 person ，获得 person 的 myName 属性
			// 通过一个个参数传递
			getInfo.call(newPerson,1,2);
			// 利用 apply 函数，传入指定上下文 person ，获得 person 的 myName 属性
			// 通过数组方式传递参数
			getInfo.apply(newPerson,[1,2]);

			// ES 5 的新增函数 bind，也是传入指定上下文，但不会马上执行，只是初始化
			var getInfoBind = getInfo.bind(newPerson,1,2);
			// 现在才执行
			console.log('bind to run');
			getInfoBind();



			// 1，简单版：通过 apply 实现 bind
			// 直接在函数原型链上新增函数
			Function.prototype.myBind = function(){
				// 保存当前上下文
				var self = this;
				// 第一个参数是传入的对象，其他是输入参数
				// 弹出第一个参数
				var obj = Array.prototype.shift.call(arguments);
				// 获取剩余参数
				var args = Array.prototype.slice.call(arguments);
				// 返回 bind 函数
				return function(){
					// 这里的 arguments 跟上面的不一样，是返回函数自身的 arguments
					var allArgs = args.concat(Array.prototype.slice.call(arguments));
					// 当被执行时，绑定对于上下文
					return self.apply(obj,allArgs);
				}
			}

			console.log('myBind to run');
			// 使用 apply 实现的 bind 函数，初始化
			var getInfoMyBind = getInfo.myBind(newPerson,1,2);
			// 执行
			getInfoMyBind();



			// 2，升级版：考虑到 bind 作为构造函数的返回情况
			// 测试上面的 bind 的实现，发现会丢失构造函数的属性
			function Student(){
				this.school = 'Tsinghua';
				console.log('school : ' + this.school + '; name : ' + this.name);
				
			}

			var Lose = {
				name: 'Lose'
			}
			var StudentFunc = Student.myBind(Lose);
			console.log('StudentFunc')
			// 普通函数执行方式
			StudentFunc();


			var Jack = {
				name: 'Jack'
			}
			console.log('StudentConstructor')
			var StudentConstructor = Student.myBind(Jack);
			// 构造函数调用
			var s2 = new StudentConstructor();			
			// 输出 undefined，因为丢失了原型链
			console.log(s2.school);
			console.log(s2.name);


			Function.prototype.myBindNew = function(){
				// 保存当前上下文
				var self = this;
				// 第一个参数是传入的对象，其他是输入参数
				// 弹出第一个参数
				var obj = Array.prototype.shift.call(arguments);

				// 获取剩余参数
				var args = Array.prototype.slice.call(arguments);

				// 如果被构造函数调用 bind ，必须返回包含构造函数原型链的函数

				// 保存原来构造函数的原型链
				var func = function() {};
				// 当Function.prototype.bind()直接调用时，此时的 self，指向 Function.prototype
				// self.prototype(即Function.prototype.prototype)为undefined
				// 被其他函数初始化，self.prototype 就是函数的原型对象，保存下来
				if (self.prototype) {
					func.prototype = self.prototype; 
				}

				// 将要返回的函数
				var funcCons = function() {
					// 保存返回函数被执行时的上下文
					// 普通函数执行时，上下文是 window
					// 如果上下文就是当前的返回函数，说明是通过 new 来执行的
					var _self = this;

					// 普通函数执行，传入之前指定的上下文 obj
					// 如果是 new 方式执行，那么忽略掉上下文 obj，直接使用指定构造函数为上下文
					var newObj = _self instanceof funcCons ? _self : obj;
					// 获取返回函数执行时的所有参数
					var allArgs = args.concat(Array.prototype.slice.call(arguments));

					return self.apply(newObj,allArgs);
				};

				// 将返回函数的原型链指向之前保存的原型链
				funcCons.prototype = new func();


				// 返回 bind 函数
				return funcCons;
			}


			var Steven = {
				name: 'Steven'
			}
			console.log('myBindNew')

			console.log('StudentFuncNew')
			var StudentFuncNew = Student.myBindNew(Steven);
			// 普通函数执行方式
			StudentFuncNew();


			console.log('StudentConstructor')
			var StudentConstructorNew = Student.myBindNew(Steven);
			// 构造函数调用
			var s3 = new StudentConstructorNew();
			// 能够正常输出构造函数的属性
			console.log(s3.school);
			// bind 的实现遇到构造函数时，直接绑定构造上下文
			// 所以 bind 传入的上下文，被忽略了，所以也就无法获取传入对象的属性啦
			// 这里依然输出 undefined
			console.log(s3.name);

```