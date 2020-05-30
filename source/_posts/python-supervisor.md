---
title: supervisor
date: 2020-05-30 15:39:48
tags: Python
---

> supervisor

<!-- more -->

## 一. 作用
supervisor是用Python开发的一个client/server服务，是Linux/Unix系统下的一个进程管理工具。可以很方便的监听、启动、停止、重启一个或多个进程。用supervisor管理的进程，当一个进程意外被杀死，supervisor监听到进程死后，会自动将它重启，很方便的做到进程自动恢复的功能，不再需要自己写shell脚本来控制。

## 二. 类似工具

##### 1. Linux 自带的 systemd 的 Restart = always
##### 2. Node 的进程管理工具 pm2
