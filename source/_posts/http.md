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


![](/example/img/HTTP-layers.png)



##### HTTP 报文基本格式
![](/example/img/http-protocol.png)



### 1. 请求报文
![](/example/img/http-request.jpg)


### 2. 返回报文
![](/example/img/http-response.png)


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
![](/example/img/tcp.jpg)


- 第一次 HTTP 请求，初始化链接时，进行 TCP 三次握手
![](/example/img/tcp-one.png)


- 第二次 HTTP 请求，共享刚才的 TCP 链接，节省了初始化链接的时间
![](/example/img/tcp-two.png)



## 三，头部信息

### 头部信息格式 name:value1,value2,value3 
```
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
```

### 1. 请求头部 Request Headers

名称|解释
-|-
Accept|指定客户端能够接收的内容类型。
Accept-Charset|浏览器可以接受的字符编码集。
Accept-Encoding|指定浏览器可以支持的web服务器返回内容压缩编码类型。
Accept-Language|浏览器可接受的语言。
Accept-Ranges|可以请求网页实体的一个或者多个子范围字段。
AuthorizationHTTP|授权的授权证书。
Cache-Control|指定请求和响应遵循的缓存机制。
Connection|表示是否需要持久连接。（HTTP 1.1默认进行持久连接）
CookieHTTP|请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。
Content-Length|请求的内容长度。
Content-Type|请求的与实体对应的MIME信息。
Date|请求发送的日期和时间。
Expect|请求的特定的服务器行为。
From|发出请求的用户的Email。
Host|指定请求的服务器的域名和端口号。
If-Match|只有请求内容与实体相匹配才有效。
If-Modified-Since|如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码。
If-None-Match|如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变。
If-Range|如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。
If-Unmodified-Since|只在实体在指定时间之后未被修改才请求成功。
Max-Forwards|限制信息通过代理和网关传送的时间。
Pragma|用来包含实现特定的指令。
Proxy-Authorization|连接到代理的授权证书。
Range|只请求实体的一部分，指定范围。
Referer|先前网页的地址，当前请求网页紧随其后,即来路。
TE|客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息。
Upgrade|向服务器指定某种传输协议以便服务器进行转换（如果支持。
User-Agent|内容包含发出请求的用户信息。
Via|通知中间网关或代理服务器地址，通信协议。
Warning|关于消息实体的警告信息


### 2. 响应头部 Response Headers

名称|解释
-|-
Accept-Ranges|表明服务器是否支持指定范围请求及哪种类型的分段请求。
Age|从原始服务器到代理缓存形成的估算时间（以秒计，非负）。
Allow|对某网络资源的有效的请求行为，不允许则返回405。
Cache-Control|告诉所有的缓存机制是否可以缓存及哪种类型。
Content-Encodingweb|服务器支持的返回内容压缩编码类型。。
Content-Language|响应体的语言。
Content-Length|响应体的长度。
Content-Location|请求资源可替代的备用的另一地址。
Content-MD5|返回资源的MD5校验值。
Content-Range|在整个返回体中本部分的字节位置。
Content-Type|返回内容的MIME类型。
Date|原始服务器消息发出的时间。
ETag|请求变量的实体标签的当前值。
Expires|响应过期的日期和时间。
Last-Modified|请求资源的最后修改时间。
Location|用来重定向接收方到非请求URL的位置来完成请求或标识新的资源。
Pragma|包括实现特定的指令，它可应用到响应链上的任何接收方。
Proxy-Authenticate|它指出认证方案和可应用到代理的该URL上的参数。
refresh|应用于重定向或一个新的资源被创造，在5秒之后重定向（由网景提出，被大部分浏览器支持）
Retry-After|如果实体暂时不可取，通知客户端在指定时间之后再次尝试。
Serverweb|服务器软件名称。
Set-Cookie|设置Http Cookie。
Trailer|指出头域在分块传输编码的尾部存在。
Transfer-Encoding|文件传输编码。
Vary|告诉下游代理是使用缓存响应还是从原始服务器请求。
Via|告知代理客户端响应是通过哪里发送的。
Warning|警告实体可能存在的问题。
WWW-Authenticate|表明客户端请求实体应该使用的授权方案。



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
503|Service Unavailable 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中
504|Gateway Time - out|充当网关或代理的服务器，未及时从远端服务器获取请求
505|HTTP Version not supported|服务器不支持请求的HTTP协议的版本，无法完成处理
