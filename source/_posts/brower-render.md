---
title: 浏览器输入地址后发生的事件
date: 2019-11-07 08:29:27
tags: JavaScript
---

> 在浏览器输入地址后，主要经历了域名 DNS 解释，TCP 三次握手，HTTP 请求，HTTP 响应，解析 DOM 和 CSSOM，渲染和绘制页面。
> 为了优化前端页面，需要对整个过程进行性能监控，目前主流浏览器提供了 window.performance.timing 记录每个步骤的时间戳。
> 在最新的规范中，对以上过程进行分类，并推荐使用 PerformanceResourceTiming 和 PerformanceNavigationTiming 来记录时间戳。

<!-- more -->

## 一. 总体流程概览
### 1. 浏览器输入地址，经过一系列事件后，把页面呈现给用户，主要过程如下图
![](/img/2019/browser-two.png)


### 2. 在最新规范中，将上述过程进行分类，如下图所示
![](/img/2019/browser-three.png)

### 3. 从浏览器分析每个流程

![](/img/2020/page_build_1.png)

- Queueing 
HTTP 请求的队列等待时间，HTTP 底层采用 TCP 协议，同一个时间段内一个条 TCP 通道只允许一个 HTTP 请求，Chrome 允许最多打开 6 个 TCP，因此最多并发 6 个 HTTP 请求。 HTTP2 提供了 Multiplexing 多路传输特性，可以在一个 TCP 连接中同时完成多个 HTTP 请求，但是目前还没开始普及。


- DNS Lookup
域名解析，将 domain 解析成 IP 地址

- Initial connection
浏览器与服务器建立连接，包括 TCP 三次握手，以及 SSL 验证

- Request sent
当连接建立后，浏览器开始向服务器发起请求

- Waiting (TTFB,Time To First Byte)
等待服务器返回第一个字节，此过程耗时主要是服务器内部处理请求的时间

- Content Download
浏览器收到服务器完整的返回内容，这个时间取决于返回内容大小，以及网络情况。

- DOMContentLoaded
浏览器开始处理服务器返回的内容，这个时间是 HTML 的 DOM 解析并加载完成，但是异步下载的 JS，图片可能还在进行中。

- Load
此时浏览器已经渲染完页面，异步下载的JS，图片都完成了。


## 二. 每个事件点的解析


### 1. 每个事件点

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


### 2. 主要统计指标
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




    // 用户感知指标

    // 整体时间，用户从浏览器输入地址到整个页面展示完毕的时间
    logEventBetweenTime('navigationStart','loadEventEnd');

    // 白屏时间，浏览器卸载旧页面出现白屏，到新页面内容出现的时间
    // 此时正在解析 dom，展示出第一个dom
    logEventBetweenTime('fetchStart','domLoading');

    // 首屏时间，浏览器卸载旧页面出现白屏，到首屏绘制完毕的时间节点；
    logEventBetweenTime('fetchStart','loadEventStart');


    // 细分指标
    
    // 重定向的时间，比如 http://example.com 和 http://www.example.com/ 之间的跳转
    logEventBetweenTime('redirectStart','redirectEnd');

    // DNS 域名解析时间
    logEventBetweenTime('domainLookupStart','domainLookupEnd');

    // TCP 建立连接完成握手的时间，包括 SSL
    logEventBetweenTime('connectStart','connectEnd');

    // TTFB 即 Time To First Byte，浏览器发起请求到读取页面第一个字节的时间
    logEventBetweenTime('requestStart','responseStart');

    // 服务器返回内容的时间
    logEventBetweenTime('responseStart','responseEnd');

    // DOM 解析耗时
    logEventBetweenTime('responseEnd','domInteractive');
};

```

### 3. 页面事件节点数据上报方式
- XMLHttpRequest，异步有可能会被浏览器忽略掉，同步会阻碍页面卸载和跳转。
- new Image，依然会阻碍页面卸载和跳转，而且只能用 get 方式。
- navigator.sendBeacon，浏览器空闲时才异步地向服务器发送数据，同时不会延迟页面的卸载，还能保证数据传输可靠性。

```js
window.addEventListener('unload', logData, false);

function logData() {
    navigator.sendBeacon("/log", analyticsData);
}
```


## 三. 主要过程详细介绍


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


## 四. 模拟实现重点事件函数

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


## 五. 前端页面加载优化方法



### 1. 减少 HTTP 请求次数
如果超过了浏览器并发 TCP 连接数，那么 HTTP 请求需要排队执行；多次 HTTP 请求的网络往返时间，肯定会比一次 HTTP 请求耗时更长。

- 合并 HTTP 请求
多个 JS 文件打包成一个。
多个 CSS 文件打包成一个。

- 图片传输特殊处理
图片转换成 Base64 编码传输
多个图片合并成一个雪碧图

- 合理使用缓存
使用 cach-control 或 expires 缓存时，如果没有期，不会向服务器发送请求；
如果过期了，会使用 last-modified 或 etag 这类协商缓存，向服务器发送请求，如果资源没有变化，则服务器返回304响应，浏览器继续从本地缓存加载资源；
如果资源更新了，则服务器将更新后的资源发送到浏览器，并返回200响应。


- 减少 URL 重定向
避免使用重定向，服务器 URL 匹配规则兼容 http://www.example.com 和 http://www.example.com
如果确实使用重定向，比如 http 跳到 https，尽量使用永久重定向301，因为临时重定向302会导致浏览器依然会发起一次 http 请求到服务器，永久重定向301则在浏览器直接发起 https 请求。




### 2. 优化网络传输过程

- DNS预解析和缓存
当浏览器从第三方服务跨域请求资源的时候，这个第三方的跨域域名需要被解析为一个IP地址，这个过程就是DNS解析，DNS预解析和缓存可以用来减少这个过程的耗时。
dns-prefetch 仅对跨域域上的DNS查找有效，因此请避免将其用于当前访问的站点。这是因为当浏览器解析到当前Link元素内容时，您站点域名的IP已经被解析。

```html
<link rel="dns-prefetch" href="https://fonts.googleapis.com/">
```

- 使用 CDN
请求资源就近访问原则，减少网络物理链路，降低网络传输时耗。

- 降低资源大小
JS，CSS，图片文件压缩
HTTP 协议开启 gzip 压缩


- 尽量使用 TCP 长连接，避免重复三次握手
HTTP 协议打开 keep-alive

- 增加机房带宽

### 3. 服务器处理请求优化

- 使用缓存页面，避免重复生成 HTML 页面

- 数据库优化，建立合理索引，读写分离，数据缓存

### 4. 页面资源加载顺序优化

- 优先加载 CSS，因为浏览器首先解析 HTML 的 DOM 和 CSS 的 CSSOM，因此放到前面 <head> 加载。

- JS 文件放到最后加载，JS 加载和解析将会阻塞浏览器解析 DOM，所以放到最后 </body> 才加载。

- 按需加载资源，加快首页打开的速度

### 5. 页面内容编写优化

- html 编写严格按照标准，必须包含开始和闭合标签，减少 dom 解析时间。

- 减少页面的重排和重绘，尽量不要通过 JS 多次修改 DOM 树。

- 减少深层次的 DOM 遍历，尽量不要使用太复杂的 CSS 选择器和表达式。

- 使用函数节流（throttle）或函数去抖（debounce），限制某一个方法的频繁触发。

- 少用 table 布局页面，因为要等待整个 table 内容加载才会显示。

- 添加 loading 等待圈，用户体验更好