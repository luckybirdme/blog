---
title: 浏览器输入地址后发生了什么
date: 2019-11-07 08:29:27
tags: JavaScript
---

> 在浏览器输入地址后，主要经历了域名 DNS 解释，TCP 三次握手，HTTP 请求，HTTP 响应，解析 DOM 和 CSSOM，渲染和绘制页面。
> 为了优化前端页面，需要对整个过程进行性能监控，目前主流浏览器提供了 window.performance.timing 记录每个步骤的时间戳。
> 在最新的规范中，对以上过程进行分类，并推荐使用 PerformanceResourceTiming 和 PerformanceNavigationTiming 来记录时间戳。

<!-- more -->

## 一，概览
### 1. 浏览器输入地址，经过一系列事件后，把页面呈现给用户，主要过程如下图
![](/img/2019/browser-two.png)


### 2. 在最新规范中，将上述过程进行分类，如下图所示
![](/img/2019/browser-three.png)


### 3. 每个事件点的解析如下：
事件|解析
-|-
navigationStart|当卸载提示在同一浏览上下文中的上一个文档终止时. 如果没有以前的文档，则此值将与PerformanceTiming.fetchStart相同.
unloadEventStart|当引发了unload事件时，指示窗口中先前的文档开始卸载的时间. 如果没有先前的文档，或者先前的文档或所需的重定向之一不是同一来源，则返回的值为0 .
unloadEventEnd|unload事件处理程序完成时. 如果没有先前的文档，或者先前的文档或所需的重定向之一不是同一来源，则返回的值为0 .
redirectStart|当第一个HTTP重定向开始时. 如果没有重定向，或者其中一个重定向源不相同，则返回值为0 .
redirectEnd|当最后一个HTTP重定向完成时，即已收到HTTP响应的最后一个字节. 如果没有重定向，或者其中一个重定向源不相同，则返回值为0 .
fetchStart|当浏览器准备好使用HTTP请求获取文档时. 此刻在检查任何应用程序缓存之前
domainLookupStart|当域查找开始时. 如果使用持久连接，或者信息存储在缓存或本地资源中，则该值将与PerformanceTiming.fetchStart相同.
domainLookupEnd|域查找完成后. 如果使用持久连接，或者信息存储在缓存或本地资源中，则该值将与PerformanceTiming.fetchStart相同.
connectStart|TCP 握手开始，当打开连接的请求发送到网络时. 如果传输层报告错误，并且连接建立再次开始，则给出最后的连接建立开始时间. 如果使用持久连接，则该值将与PerformanceTiming.fetchStart相同.
secureConnectionStart|SSL 安全连接握手开始时. 如果不请求此类连接，则返回0 .
connectEnd|TCP 建立链接，打开连接网络时. 如果传输层报告错误，并且连接建立再次开始，则给出最后的连接建立结束时间. 如果使用持久连接，则该值将与PerformanceTiming.fetchStart相同. 当所有安全连接握手或SOCKS身份验证终止时，连接被视为已打开.
requestStart|当浏览器发送请求以从服务器或缓存中获取实际文档时. 如果在请求开始后传输层失败，并且连接重新打开，则此属性将设置为与新请求对应的时间.
responseStart|当浏览器从服务器从缓存或本地资源接收到响应的第一个字节时.
responseEnd|当浏览器收到响应的最后一个字节时，或者如果第一次发生则关闭连接时，则来自服务器，缓存或本地资源.
domLoading|解析器开始工作时，即其Document.readyState更改为'loading'并且引发了相应的readystatechange事件.
domInteractive|解析器完成对主文档的工作时，即其Document.readyState更改为'interactive'并且引发了相应的readystatechange事件，但是内嵌资源（比如外链css、js等）还未加载的时间点，下一步会触发 jquery 的 $(document).ready()
domContentLoadedEventStart|在解析器发送 DOMContentLoaded 事件之前，即在解析后立即需要执行的所有脚本之后，此时还在加载css，js中。
domContentLoadedEventEnd|在所有需要尽快执行的脚本（无论是否按顺序执行）之后，此刻用户可以对页面进行操作。
domComplete|解析器完成对主文档的工作时，即其Document.readyState更改为'complete'并且引发了相应的 readystatechange 事件，此时外部资源css，js等已经加载完毕，用户此时可操作页面，下一步触发 onload 事件
loadEventStart|为当前文档发生onload事件的时间. 如果尚未发送此事件，则返回0.
loadEventEnd|当onload事件处理程序终止时，即加载事件完成时. 如果此事件尚未发送或尚未完成，则返回0.


### 4. 主要统计时间
```js
var performance = function(){

    var t = window.performance.timing;

    var logEventTime=function(name){
        console.log(name,t[name]);
    }

    var logEventBetweenTime=function(event1,event2){
        console.log(event1,event2,t[event2]-t[event1]);
    }

    logEventTime('navigationStart');
    logEventTime('unloadEventStart');
    logEventTime('unloadEventEnd');
    logEventTime('redirectStart');
    logEventTime('redirectEnd');
    logEventTime('fetchStart');
    logEventTime('domainLookupStart');
    logEventTime('domainLookupEnd');
    logEventTime('connectStart');
    logEventTime('secureConnectionStart');
    logEventTime('connectEnd');
    logEventTime('requestStart');
    logEventTime('responseStart');
    logEventTime('responseEnd');
    logEventTime('domLoading');
    logEventTime('domInteractive');
    logEventTime('domContentLoadedEventStart');
    logEventTime('domContentLoadedEventEnd');
    logEventTime('domComplete');
    logEventTime('loadEventStart');
    logEventTime('loadEventEnd');




    // 主要体验指标时间

    // 整体时间，用户从浏览器输入地址到整个页面展示完毕的时间
    logEventBetweenTime('navigationStart','loadEventEnd');

    // 白屏时间，浏览器卸载旧页面出现白屏，到新页面内容出现的时间
    // 此时正在解析 dom，展示出第一个dom
    logEventBetweenTime('fetchStart','domLoading');

    // 操作时间，浏览器卸载旧页面出现白屏，到用户可以点击节点
    logEventBetweenTime('fetchStart','domContentLoadedEventEnd'); 

    // 首屏时间，浏览器卸载旧页面出现白屏，到首屏绘制完毕的时间节点；
    logEventBetweenTime('fetchStart','loadEventStart');

    // TTFB 即 Time To First Byte，读取页面第一个字节的时间
    // 用户拿到你的资源占用的时间，加异地机房了么，加CDN 处理了么？加带宽了么？加 CPU 运算速度了么？
    logEventBetweenTime('navigationStart','responseStart');


    
    // 细分事件的时间

    // 卸载页面的时间
    logEventBetweenTime('unloadEventStart','unloadEventEnd');

    // 重定向的时间，比如，http://example.com/ 就不该写成 http://example.com
    logEventBetweenTime('redirectStart','redirectEnd');

    // 域名解析时间
    logEventBetweenTime('domainLookupStart','domainLookupEnd');

    // TCP 建立连接完成握手的时间，包括 SSL
    logEventBetweenTime('connectStart','connectEnd');

    //服务器处理请求时间，代码逻辑是不是太复杂了，数据库索引建立了吗？
    logEventBetweenTime('requestStart','responseStart');

    // http 响应返回内容的时间，内容经过 gzip 压缩了么，静态资源 css/js 等压缩了么？
    logEventBetweenTime('responseStart','responseEnd');


};

```

### 5. 页面监控数据上报通过
- XMLHttpRequest，异步有可能会被浏览器忽略掉，同步会阻碍页面卸载和跳转。
- new Image，依然会阻碍页面卸载和跳转，而且只能用 get 方式。
- navigator.sendBeacon，浏览器空闲时才异步地向服务器发送数据，同时不会延迟页面的卸载，还能保证数据传输可靠性。

```js
window.addEventListener('unload', logData, false);

function logData() {
    navigator.sendBeacon("/log", analyticsData);
}
```


## 二，主要过程


### 1. DNS 解析：domainLookupStart, domainLookupEnd


#### 域名解析 Domain name resolution，也叫 DNS 解析，就是根据用户输入的域名，查找目的 IP，查询顺序如下：
- 浏览器 DNS 缓存
- 本机电脑 host 文件
- 所在网络路由 DNS 缓存
- 所在区域 DNS 服务器
- Root server 域名服务器


#### 客户端前往域名服务器查询，一般采用递归查询和迭代查询结合的方式，如下图所示：

![](/img/2019/dns.png)


#### 域名解析通过 53 端口，传输协议有UDP和TCP，但更多是使用高效的UDP，并借助缓存加快解析速度：
- UDP，无需连接，效率更快；安全性低，字节小于512；
- TCP，建立安全连接，传输数据大；建立连接耗时长。



### 2. TCP 连接：connectStart, connectEnd


- 浏览器是通过 HTTP 协议与服务端交互，而 HTTP 底层是通过 TCP 传输报文。
- TCP 三次握手，建立链接，如果是 SSL 链接，会加上密匙交互和加密过程。

![](/img/2019/http-tcp.jpg)


### 3. HTTP request 和 response : requestStart, responseStart, responseEnd
- TCP 建立连接后，开始发送 HTTP request 报文，格式如下：

![](/img/2019/http-request.png)


- HTTP response 报文格式如下，其中响应内容是浏览器页面将要展示的内容：

![](/img/2019/http-response.png)


### 4. HTML 解析，渲染和绘制 : domLoading, domInteractive, domContentLoadedEventStart, domContentLoadedEventEnd, domComplete
- HTTP 响应内容包括了 HTML，CSS 等，浏览器开始处理这些内容。

![](/img/2019/html-css.png)



### 5. JS 的 onload 事件 ： loadEventStart, loadEventEnd
- 在 HTML 解析完 DOM 后，而且其他图片，JS等资源加载完后，会触发 onload 事件

```js
window.onload = function(){
	console.log("load event detected!");
};
```


## 三，关键函数

```js
// dom 解析完，在 domInteractive 事件后执行
$(document).ready(function() {
    // body...
    console.log('2,document,ready');
});

// dom 解析完，在 domContentLoadedEventStart 事件后执行
document.addEventListener('DOMContentLoaded', (event) => {
    console.log('3,document,DOMContentLoaded');
});


document.onreadystatechange=function(){
    // dom 正在解析中，处于 domLoading 状态
    if(document.readyState === 'loading'){
        console.log('0,document,loading');
    }
    // 页面事件处于 domInteractive，已经完成 dom 解析，外部资源还没加载完成
    // 模拟 DOMContentLoaded/ jquery ready
    if(document.readyState==='interactive'){
        console.log('1,document,interactive,to do jq ready');
    }

    // 页面事件处于 domComplete，在文档中的所有对象都在DOM中，所有图片，脚本，链接以及子框都完成了装载
    // 模拟 load 事件
    if(document.readyState==='complete'){
        console.log('4,document,complete, to do window onload');
    }
}

// 页面事件 loadEventStart 后发生 onload 事件
window.onload=function(){
    console.log('5,window,onload');
}
```