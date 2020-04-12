---
title: ajax
date: 2020-01-29 09:26:30
tags: JavaScript
---

> ajax

<!-- more -->

AJAX 是 Asynchronous JavaScript And XML 的首字母缩写，底层利用 XMLHttpRequest 对象，在浏览器不刷新页面的情况下，跟服务器进行交互。


```js


var xmlhttp;
// 创建XMLHttpRequest
if (window.XMLHttpRequest){ 
    // code for IE7+, Firefox, Chrome, Opera, Safari
    xmlhttp=new XMLHttpRequest();
}else{  
    // code for IE6, IE5
    xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
}
// 设置超时，单位秒；
xmlhttp.timeout = 10000; 
// 超时处理函数
xmlhttp.ontimeout = function (e) {
  // XMLHttpRequest timed out. Do something here.
  alert('亲，网络塞车了，先听首歌吧');
};

// 请求返回处理
xmlhttp.onreadystatechange = function() {
        // 读取对象状态
    if (xmlhttp.readyState == XMLHttpRequest.DONE) {
        // 读取返回状态和结果，200表示http请求成功
        if (xmlhttp.status == 200) {
            // 获取返回数据
            var response =  eval('('+xmlhttp.responseText+')');
            
        } else {
            // 异常处理
            alert('亲，汽车抛锚了，正在维修中');
        }
    }

} 

// 请求方式POST/GET
var method = 'POST';
//请求地址url
var url = 'http://localhost/api/';
// true（异步）或 false（同步）
var async = true;               
xmlhttp.open(method,url,async);
// 设置头部信息
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
// 发起请求，并传递参数
xmlhttp.send('key1=val1&key2=val2');



```