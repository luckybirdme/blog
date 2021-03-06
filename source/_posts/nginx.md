---
title: nginx 简介
date: 2020-04-11 21:40:36
tags: NGINX
---
> web 服务器
> 负载均衡(反向代理)

<!-- more -->

## 一. 作用

##### 1. 支持 HTTP 协议转发，即第七层转发
- 处理静态文件，索引文件以及自动索引；
- 反向代理，简单的负载均衡和容错；
- FastCGI，比如PHP-FPM；
- SSL 和 TLS 支持；
- 支持文件压缩，缓存；
- 提供IP拦截，链接限制；
- 支持 keep-alive 和管道连接；
- 可定制的访问日志，日志写入缓存，以及快捷的日志回卷；
- 4xx-5xx 错误代码重定向；
- 基于 PCRE 的 rewrite 重写模块；
- 基于客户端 IP 地址和 HTTP 基本认证的访问控制；
- 支持热部署，即 reload 配置文件

##### 2. 支持邮件协议 IMAP/POP3 转发 

##### 3. NGINX 1.9 版本以上支持 TCP/UDP 协议转发，即第四层转发


## 二. 应用场合

##### 1. 静态服务器，比如图片，视频服务，js，css等，并发 几万以上。
##### 2. 动态服务，fastcgi 的方式运行脚本，比如 PHP，并发在500-1500。
##### 3. 反向代理，负载均衡，转发服务到多台机器，可以直接用nginx做代理。
##### 4. 缓存服务，类似 SQUID，VARNISH。

## 三. 特性对比


分类|Apache | NGINX | LVS
-|-|-|-
转发协议层| http(七层)|专注http(七层)，也支持TCP/UDP(四层)|专注 TCP/UDP(四层)
IO模式|select模式，每次遍历所有文件句柄 fd，性能低，而且fd数量限制跟系统有关，32位是1024个|epoll模式， 只对活跃的文件句柄 fd 扫描，可满足高并发IO，而且没有fd数量限制|epoll 模式
静态文件转发|较弱|高效|不支持
并发性能|较弱|高效|性能最好，因为转发在第四层，没有流量产生
部署|简单|配置简单，对网络稳定性的依赖非常小，理论上能ping通就能进行负载功能|相对复杂，而且要求机器在同一局域网




## 四. 遇到问题


##### 1. 无法保存 Session
第一次http访问转发A服务器后，下一次请求又突然转到B服务器上。这个时候与A服务器建立的Session，B服务器是不知道的。
解决方案：
- Session凭据保存共用的数据源中，比如 Redis，以台多台机器共享
- 通过 nginx ip_hash 保持同一IP的请求都是指定到固定的一台服务器

##### 2. 对后端服务器的健康检查，只支持通过端口来检测，不支持通过url来检测。




