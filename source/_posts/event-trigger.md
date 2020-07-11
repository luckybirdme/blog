---
title: JS 事件捕获和冒泡
date: 2019-11-16 16:20:18
tags: JavaScript
---

> 事件捕获阶段，从顶层节点开始，一层层往下，直到触发事件的节点。
> 事件冒泡阶段，从触发事件的节点开始，一层层向上，直到顶层节点。

<!-- more -->


## 一，JS 事件流
### JS 事件流如下图所示，左边为事件捕获阶段，右边为事件冒泡阶段
![](/img/2019/event-trigger.png)


### addEventListener(type, listener, useCapture),第三个参数 useCapture 规定事件触发时机
- false,表示 listener 事件触发在冒泡阶段，这是默认值。
- true,表示 listener 事件在捕获阶段就触发了。


## 二，事件捕获和冒泡时触发 listener

```html
<!DOCTYPE html>
<html>
<body id="body">
    <div id="div">
        <button type="button" id="button">click me</button>
    </div>
    <script type="text/javascript">
        var body = document.getElementById('body');
        var div = document.getElementById('div');
        var button = document.getElementById('button');

        document.addEventListener('click',function(e) {
           console.log('document');
        },true)

        body.addEventListener('click',function(e){
            console.log('body');
        });

        div.addEventListener('click',function(e){
            console.log('div');
        },true);

        button.addEventListener('click',function(e){
            console.log('button');
        });
    </script>
</body>
</html>
```


## 三，阻止事件

### 1. stopPropagation, 阻止捕获和冒泡阶段中当前事件的进一步传播。
```js
document.addEventListener('click',function(e) {
   console.log('document');
})

body.addEventListener('click',function(e){
    console.log('body');
});

div.addEventListener('click',function(e){
    console.log('div');
    console.log('stop');
    e.stopPropagation();
});

button.addEventListener('click',function(e){
    console.log('button');
});

/* output
button
div
stop
*/
```


### 2. preventDefault, 阻止系统默认事件，比如 a 标签点击会跳转到 href 指定地址。通过 preventDefault 可以拦截系统默认的跳转，但是事件依然会继续传播到下一个节点。

```js
document.addEventListener('click',function(e) {
   console.log('document');
})

body.addEventListener('click',function(e){
    console.log('body');
});

div.addEventListener('click',function(e){
    console.log('div');
});
a.addEventListener('click',function(e){
    console.log('a');
    e.preventDefault();
});
/*
a
div
body
document
*/
```

## 四，委托事件

### 1. 存在问题 
- 如果需要绑定多个 li 的点击事件，那需要遍历每个 li 进行绑定，相对较慢；而且越多绑定事件，JS内存消耗也大
- 如果页面初始化生成后，对于新增的 li 是没有绑定事件的，需要人工再次绑定

```html

<ul id="ul1">
    <li>111</li>
    <li>222</li>
    <li>333</li>
    <li>444</li>
</ul>

```


```js
window.onload = function(){
    var oUl = document.getElementById("ul1");
    var aLi = oUl.getElementsByTagName('li');
    for(var i=0;i<aLi.length;i++){
        aLi[i].onclick = function(){
            alert(123);
        }
    }
}

```

### 2. 委托事件
- 利用事件冒泡的机制，从底层向高层冒泡，所以可以将子节点的事件，统一冒泡到父节点进行处理，即委托给父节点的处理事件。
- 因为是绑定了父节点，父节点在初始化时就存在并绑定了事件，所以新增的子节点也会自动绑定事件，无需再次人工绑定了。

```js

window.onload = function(){
　　var oUl = document.getElementById("ul1");
　　oUl.onclick = function(ev){
　　　　var ev = ev || window.event;
　　　　var target = ev.target || ev.srcElement;
     // 避免父节点本身点击事件的影响
　　　　if(target.nodeName.toLowerCase() == 'li'){
　 　　　　　　 alert(123);
　　　　　　　  alert(target.innerHTML);
　　　　}
　　}
}
```


