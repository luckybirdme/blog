---
title: 前端页面安全
date: 2019-11-16 10:46:52
tags: html
---

> 前端常见的安全隐患包括：SQL注入，XSS，CSRF，点击劫持

<!-- more -->


## 一，SQL 注入
##### 伪造特殊的输入参数达到执行特殊 SQL 的目的，可能是脱库（获取敏感数据），也能直接删除敏感数据。
#### 1. 原理：
前端输入用户 id 进行搜索
```html
https://www.example.com?id=1
```
后台根据 id 搜索
```sql
select * from user where id = 1
```
使用特殊输入进行 SQL 注入
```html
https://www.example.com?id=1 or 1=1
```
后台对输入没有校验，直接搜索出全部用户信息
```sql
select * from user where id = 1 or 1=1
```


#### 2. 防范：
##### 对输入参数进行校验，过滤，转义等
比如，输入参数添加引号，后台查询 SQL 变成
```sql
select * from user where id = '1 or 1=1'
```


## 二，XSS
##### 跨站脚本攻击（Cross Site Scripting），攻击者通过注入非法的 html 标签或者 javascript 代码，从而当用户浏览该网页时，会执行非法的代码，达到控制用户浏览器的目的。
### 1. 原理：
用户评论内容保存到数据库
```html
<textarea >
你将会看到这个弹框 <script>alert("Hello")</script>
</textarea>
```
后台直接保存了包含 js 代码的内容
```sql
insert into comment (content) values('你将会看到这个弹框 <script>alert("Hello")</script>') 
```
用户查看评论的页面会执行这段 js 代码
```html
<div>
你将会看到这个弹框 <script>alert("Hello")</script>
</div>
```

### 2. 分类：
- 反射型XSS也被称为非持久性XSS。XSS代码出现在URL中，最后输入提交到服务器，服务器解析后在响应内容中出现这段XSS代码，最后浏览器解析执行。
- 存储型XSS又被称为持久性XSS，相比反射型XSS具有更高的隐蔽性，因为它不需要用户手动触发。攻击者提交一段XSS代码后，被服务器端接收并存储，所有用户访问某个页面时都会被XSS，其中最典型的例子就是留言板。


### 3. 危害
- 利用虚假输入表单骗取用户个人信息。
- 利用脚本窃取用户的 Cookie 值，被害者在不知情的情况下，帮助攻击者发送恶意请求。


### 4. 防范：
- httpOnly: 在 cookie 中设置 HttpOnly 属性，使js脚本无法读取到 cookie 信息。
- 字符转义：对用户输入的任何内容都进行转义，包括引号、尖括号、斜杠等字符进行转义。
- CSP：Content Security Policy，设置允许浏览器加载哪些外部资源，以及内联脚本。


## 三，CSRF
##### 跨站点请求伪造（Cross-Site Request Forgeries），在用户不知情的情况下，冒充用户发起请求， 完成一些非用户意愿的事情，比如修改用户信息，发起评论等。

### 1. 原理
用户登录了网站 A，保存了网站 A 的 cookie，发起评论的代码如下
```html
<form action="http://www.a.com" method="post">
    <input name="comment" value="" type="text">
</form>
```

用户在浏览网站 B，点到了一个隐藏按钮，结果相当于带着网站 A 的cookie 向网站 A 发起了评论
```html
<form action="http://www.a.com" method="post">
    <input name="comment" value="attack content" type="hidden">
</form>
```

### 2. 区别：
- XSS 是通过伪造非法代码来执行，更多是属于代码层面。
- CSRF 是携带 cookie 伪装用户发起请求，更多属于 HTTP 协议层面。


### 3. 防范：
- 使用验证码，强制用户必须与应用进行交互，才能完成最终请求。此种方式能很好的遏制 csrf，但是用户体验比较差。
- 使用 post 进行数据修改，限制 get 使用，避免敏感信息泄露，也增加伪造成本。
- 检查 Referer，请求来源限制，此种方法成本最低，但是并不能保证 100% 有效，因为服务器并不是什么时候都能取到 Referer，而且低版本的浏览器存在伪造 Referer 的风险。
- 使用 token 验证的 CSRF 防御机制是公认最合适的方案。

### 4. token 原理：
- 后端随机产生一个 token，把这个token 保存到服务器 session 状态中，同时后端把这个token 交给前端页面。
- 前端页面提交请求时，必须把 token 加入到请求数据或者头信息中，一起传给后端。
- 后端验证前端传来的 token 与 session 是否一致，一致则合法，否则是非法请求。



## 四，点击劫持
##### click-jacking，是指利用透明的按钮做成陷阱，覆盖在 Web 页面之上，然后诱使用户在不知情的情况下，点击那个按钮。这种行为又称为界面伪装(UI Redressing)。

### 1. 原理：
在网站 A 下，通过 iframe 嵌套网站 B

```html
<div>
网站 A 内容
<iframe src="https://www.b.com">网站 B 透明，看不见</iframe>
</div>

```
用户不小心点击了在网站 A 的内容，实际上是点击了网站 B。
由于网站 B 是通过 iframe 加载的，也包括了网站 B 的 cookie，所以能发起正常的请求。
实际上，用户并不知道自己在网站 A 不小心点击一下，却是向网站 B 发了请求。



### 2. 防范：
- 使用 HTTP 返回头 X-Frame-Options 防御；DENY，表示页面不允许通过 iframe 的方式展示；SAMEORIGIN，表示页面可以在相同域名下通过 iframe 的方式展示；ALLOW-FROM，表示页面可以在指定来源的 iframe 中展示。
- 使用 JS 判断顶层窗口的域名是不是和本页面的域名一致，如果不一致就让恶意网页自动跳转到我方的网页。

```js
if (top.location.hostname !== self.location.hostname) {    
    alert("您正在访问不安全的页面，即将跳转到安全页面！")   
    top.location.href = self.location.href;
 }
 ```