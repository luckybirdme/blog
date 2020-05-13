---
title: php 和 Swoole
date: 2020-05-10 14:43:33
tags: PHP
---

> Swoole

<!-- more -->

## 一. PHP 语言的特性

- PHP 语言定位是 Web 服务的快速开发编程语言。
- PHP 语言之所以能如此成功，并不是性能强大功能多，而是可以快速开发。
- 即便是 PHP7 大幅提升了语言性能，并加入 JIT 特性，但目前为止 PHP 的优势依然是快速开发。
- 让 PHP 语言和 Java/C++/Golang 这些语言对标性能，没有太多的意义，好像让汽车和飞机比快一样。


## 二. Swoole 的使用

- 使用场景

Swoole 使 PHP 开发人员可以编写高性能高并发的 TCP、UDP、Unix Socket、HTTP、 WebSocket 等服务，让 PHP 不再局限于 Web 领域。

- 主要特点

1. 使用 C/C++ 语言编写，提供了 PHP 语言的多线程服务器。
2. 支持异步 IO 机制，类似 nodejs 的 event，通过事件循环来执行 IO 操作。
3. 支持了类似 Go 语言的协程，在用户态切换程序，比进程和线程的切换更高效的。