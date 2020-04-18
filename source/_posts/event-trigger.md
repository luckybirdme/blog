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