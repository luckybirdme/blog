---
title: nginx 链接 FastCGI 的方式
date: 2020-04-24 10:49:28
tags: NGINX
---

>  Nginx 连接 FastCGI 的方式有2种：TCP Socket 和 UNIX Domain Socket

<!-- more -->

## 一. TCP Socket
1. 采用 TCP 协议，以端口侦听的方式
2. 使用方式
```sh
# 以 PHP-FPM 为例，NGINX 转发请求 9000 端口，也就是 php-cgi 提供服务的端口
fastcgi_pass 127.0.0.1:9000;
```

## 二. UNIX Domain Socket
1. 直接以文件形式，以 stream socket 通讯
2. 使用方式
```sh
# /dev/shm 目录的内容保存在虚拟内存，读写更快
fastcgi_pass unix:/dev/shm/php-cgi.sock;
```

## 三. 区别
1. TCP Socket
- 采用 TCP 协议，相对更稳定
- NGINX 和 FastCGI 可以部署在不同机器上
- 当高并发的时候，会占用大量端口

2. UNIX Domain Socket
- 是 POSIX 操作系统里的一种组件，不需要走 TCP 层
- NGINX 和 FastCGI 必须部署在同一台机器上
- 性能略优于 TCP Socket ，但是在1w并发下相差不大