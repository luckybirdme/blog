---
title: 进程管理组件
date: 2020-05-30 15:39:48
tags: Linux
---

> 进程管理组件

<!-- more -->


## 一. 进程管理组件的使用场景

1. 服务进程一般可以通过 console 命令行启动，退出 console 时，进程也会退出，希望能通过后台运行
2. 为了尽量使用 CPU 的性能，最好服务进程数量跟 CPU 一致，而有些应用是单进程单线程的，比如 Nodejs
3. 考虑到服务多进程的可用性，肯定需要集群形式运行，而且增加实时监控，比如进程数量，内存使用情况等
4. 服务升级时，希望平滑过渡，最好是不停服地进行升级，同时支持更方便的服务进程的启停等操作


## 二. 不同应用的进程管理组件

1. supervisor
Python开发的一个client/server服务，是Linux/Unix系统下的一个进程管理工具。可以很方便的监听、启动、停止、重启一个或多个进程。用supervisor管理的进程，当一个进程意外被杀死，supervisor监听到进程死后，会自动将它重启，很方便的做到进程自动恢复的功能，不再需要自己写shell脚本来控制。

2. pm2
基于nodejs开发的进程管理器，适用于后台常驻脚本管理，同时对node网络应用有自建负载均衡功能，方便对多进程管理、监控、负载均衡：
- 内建负载均衡（使用Node cluster 集群模块、子进程，可以参考朴灵的《深入浅出node.js》一书第九章）
- 线程守护，keep alive
- 0秒停机重载，维护升级的时候不需要停机.
- 停止不稳定的进程（避免无限循环）
- 控制台检测
- 提供 HTTP API
- 远程控制和实时的接口API ( Nodejs 模块,允许和PM2进程管理器交互 )
- 现在 Linux (stable) & MacOSx (stable) & Windows (stable).多平台支持


3. PHP-FPM
PHP 脚本解析进程的管理组件，类似 supervisor
