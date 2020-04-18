---
title: http 协议简介
date: 2019-11-06 06:42:20
tags: JavaScript
---

> HTTP (超文本传输协议) 是用来在 Web 上传输文件的基础协议 ，基于TCP/IP通信协议来传递数据。
> HTTPS 是 HTTP 协议的安全版本，HTTPS 在 HTTP 上加入套接字 SSL（TLS 为 SSL 最新版）层，对网页进行加密传输。

<!-- more -->

## 一，简介
##### HTTP 是一种能够获取如 HTML 这样的网络资源的 protocol (通讯协议)。它是在 Web 上进行数据交换的基础，是一种 client-server 协议，也就是说，请求通常是由浏览器 client 发起的，资源管理服务器 server 接收请求并进行处理，然后返回响应的内容给 client。


![](/img/2019/HTTP-layers.png)



##### HTTP 报文基本格式
![](/img/2019/http-protocol.png)



### 1. 请求报文
![](/img/2019/http-request.jpg)


### 2. 返回报文
![](/img/2019/http-response.png)


## 二，特点

### 1. 简单的
设计得简单易读，方便使用。

### 2.可扩展的
HTTP headers 让协议扩展变得非常容易。只要服务端和客户端就新 headers 达成语义一致，新功能就可以被轻松加入进来。

### 3.无状态但有会话
HTTP 是无状态的，两个请求之间是没有关联。而使用 HTTP Cookies 创建一个会话让每次请求都能共享相同的上下文信息，达成共享状态。


### 4.可靠高效的连接
HTTP 底层采用可靠的 TCP 进行传输。HTTP/1.1 默认开启 Connection:keep-alive，一个 TCP 链接共享多个 HTTP 请求，避免重复握手。 Chrome 默认可并发建立 6 个 TCP 链接，处理高并发请求。

- TCP 三次握手，传输内容，断开链接
![](/img/2019/http-tcp.jpg)


- 第一次 HTTP 请求，初始化链接时，进行 TCP 三次握手
![](/img/2019/tcp-one.png)


- 第二次 HTTP 请求，共享刚才的 TCP 链接，节省了初始化链接的时间
![](/img/2019/tcp-two.png)



## 三，头部信息

### 头部信息格式 name:value1,value2,value3 
```
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
```

### 1. 请求头部 Request Headers

Header|解释|示例
-|-|-
Accept|指定客户端能够接收的内容类型|Accept: text/plain, text/html,application/json
Accept-Charset|浏览器可以接受的字符编码集。|Accept-Charset: iso-8859-5
Accept-Encoding|指定浏览器可以支持的web服务器返回内容压缩编码类型。|Accept-Encoding: compress, gzip
Accept-Language|浏览器可接受的语言|Accept-Language: en,zh
Accept-Ranges|可以请求网页实体的一个或者多个子范围字段|Accept-Ranges: bytes
Authorization|HTTP授权的授权证书|Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Cache-Control|指定请求和响应遵循的缓存机制|Cache-Control: no-cache
Connection|表示是否需要持久连接。（HTTP 1.1默认进行持久连接）|Connection: close
Cookie|HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。|Cookie: $Version=1; Skin=new;
Content-Length|请求的内容长度|Content-Length: 348
Content-Type|请求的与实体对应的MIME信息|Content-Type: application/x-www-form-urlencoded
Date|请求发送的日期和时间|Date: Tue, 15 Nov 2010 08:12:31 GMT
Expect|请求的特定的服务器行为|Expect: 100-continue
From|发出请求的用户的Email|From: user@email.com
Host|指定请求的服务器的域名和端口号|Host: www.zcmhi.com
If-Match|只有请求内容与实体相匹配才有效|If-Match: “737060cd8c284d8af7ad3082f209582d”
If-Modified-Since|如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码|If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT
If-None-Match|如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变|If-None-Match: “737060cd8c284d8af7ad3082f209582d”
If-Range|如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag|If-Range: “737060cd8c284d8af7ad3082f209582d”
If-Unmodified-Since|只在实体在指定时间之后未被修改才请求成功|If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT
Max-Forwards|限制信息通过代理和网关传送的时间|Max-Forwards: 10
Pragma|用来包含实现特定的指令|Pragma: no-cache
Proxy-Authorization|连接到代理的授权证书|Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Range|只请求实体的一部分，指定范围|Range: bytes=500-999
Referer|先前网页的地址，当前请求网页紧随其后,即来路|Referer: http://www.zcmhi.com/archives...
TE|客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息|TE: trailers,deflate;q=0.5
Upgrade|向服务器指定某种传输协议以便服务器进行转换（如果支持）|Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11
User-Agent|User-Agent的内容包含发出请求的用户信息|User-Agent: Mozilla/5.0 (Linux; X11)
Via|通知中间网关或代理服务器地址，通信协议|Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)
Warning|关于消息实体的警告信息|Warn: 199 Miscellaneous warning


### 2. 响应头部 Response Headers

Header|解释|示例
-|-|-
Accept-Ranges|表明服务器是否支持指定范围请求及哪种类型的分段请求|Accept-Ranges: bytes
Age|从原始服务器到代理缓存形成的估算时间（以秒计，非负）|Age: 12
Allow|对某网络资源的有效的请求行为，不允许则返回405|Allow: GET, HEAD
Cache-Control|告诉所有的缓存机制是否可以缓存及哪种类型|Cache-Control: no-cache
Content-Encoding|web服务器支持的返回内容压缩编码类型。|Content-Encoding: gzip
Content-Language|响应体的语言|Content-Language: en,zh
Content-Length|响应体的长度|Content-Length: 348
Content-Location|请求资源可替代的备用的另一地址|Content-Location: /index.htm
Content-MD5|返回资源的MD5校验值|Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==
Content-Range|在整个返回体中本部分的字节位置|Content-Range: bytes 21010-47021/47022
Content-Type|返回内容的MIME类型|Content-Type: text/html; charset=utf-8
Date|原始服务器消息发出的时间|Date: Tue, 15 Nov 2010 08:12:31 GMT
ETag|请求变量的实体标签的当前值|ETag: “737060cd8c284d8af7ad3082f209582d”
Expires|响应过期的日期和时间|Expires: Thu, 01 Dec 2010 16:00:00 GMT
Last-Modified|请求资源的最后修改时间|Last-Modified: Tue, 15 Nov 2010 12:45:26 GMT
Location|用来重定向接收方到非请求URL的位置来完成请求或标识新的资源|Location: http://www.zcmhi.com/archives...
Pragma|包括实现特定的指令，它可应用到响应链上的任何接收方|Pragma: no-cache
Proxy-Authenticate|它指出认证方案和可应用到代理的该URL上的参数|Proxy-Authenticate: Basic
refresh|应用于重定向或一个新的资源被创造，在5秒之后重定向（由网景提出，被大部分浏览器支持）|Refresh: 5; url=http://www.zcmhi.com/archives...
Retry-After|如果实体暂时不可取，通知客户端在指定时间之后再次尝试|Retry-After: 120
Server|web服务器软件名称|Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)
Set-Cookie|设置Http Cookie|Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1
Trailer|指出头域在分块传输编码的尾部存在|Trailer: Max-Forwards
Transfer-Encoding|文件传输编码|Transfer-Encoding:chunked
Vary|告诉下游代理是使用缓存响应还是从原始服务器请求|Vary: *
Via|告知代理客户端响应是通过哪里发送的|Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)
Warning|警告实体可能存在的问题|Warning: 199 Miscellaneous warning
WWW-Authenticate|表明客户端请求实体应该使用的授权方案|WWW-Authenticate: Basic



## 四，返回状态码

### 1. 基本分类

状态码范围|作用描述
-|-
100-199|用于指定客户端相应的某些动作
200-299|用于表示请求成功
300-399|用于已经移动的文件并且常被包含在定位头信息中指定新的地址信息
400-499|用于指出客户端的错误
500-599|用于指出服务器端的错误


### 2. 常用状态码
状态码|英文名称|中文描述
-|-|-
100|Continue|继续。客户端应继续其请求
101|Switching Protocols|切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议
200|OK|请求成功。一般用于GET与POST请求
201|Created|已创建。成功请求并创建了新的资源
202|Accepted|已接受。已经接受请求，但未处理完成
203|Non - Authoritative Information|非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本
204|No Content|无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档
205|Reset Content|重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域
206|Partial Content|部分内容。服务器成功处理了部分GET请求
300|Multiple Choices|多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择
301|Moved Permanently|永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替
302|Found|临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI
303|See Other|查看其它地址。与301类似。使用GET和POST请求查看
304|Not Modified|未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源
305|Use Proxy|使用代理。所请求的资源必须通过代理访问
306|Unused|已经被废弃的HTTP状态码
307|Temporary Redirect|临时重定向。与302类似。使用GET请求重定向
400|Bad Request|客户端请求的语法错误，服务
401|Unauthorized|请求要求用户的身份认证
402|Payment Required|保留，将来使用
403|Forbidden|服务器理解请求客户端的请求，但是拒绝执行此请求
404|Not Found|服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置 您所请求的资源无法
405|Method Not Allowed|客户端请求中的方法被禁止
406|Not Acceptable|服务器无法根据客户端请求的内容特性完成请求
407|Proxy Authentication Required|请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权
408|Request Time - out|服务器等待客户端发送的请求时间过长，超时
409|Conflict|服务器完成客户端的PUT请求是可能返回此代码，服务器处理请求时发生了冲
410|Gone|客户端请求的资源已经不存在。410不同于404，如果资源以前
411|Length Required |服务器无法处理客户端发送的不带Content-Length的请求信息
412|Precondition Failed|客户端请求信息的先决条件错误
413|Request Entity Too Large|由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry - After的响应信息
414|Request - URI Too Large|请求的URI过长（URI通常为网址），服务器无法处理
415|Unsupported Media Type|服务器无法处理请求附带的媒体格式
416|Requested range not satisfiable|客户端请求的范围无效
417|Expectation Failed|服务器无法满足Expect的请求头信息
500|Internal Server Error|服务器内部错误，无法完成请求
501|Not Implemented|服务器不支持请求的功能，无法完成请求
502|Bad Gateway|作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应
503|Service Unavailable|由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中
504|Gateway Time - out|充当网关或代理的服务器，未及时从远端服务器获取请求
505|HTTP Version not supported|服务器不支持请求的HTTP协议的版本，无法完成处理




## 五，不同版本的特点

### 1. http 1.0 和 1.1 区别

- 长连接，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

- 缓存处理，在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。

- 带宽优化及网络连接的使用，HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

- 错误通知的管理，在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

- Host头处理，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。


### 2. http 1.x 和 2.0 区别

- 新的二进制格式（Binary Format），HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。

- 多路复用（MultiPlexing），即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。

- header压缩，如上文中所言，对前面提到过HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。

- 服务端推送（server push），同SPDY一样，HTTP2.0也具有server push功能。

