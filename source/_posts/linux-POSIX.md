---
title: POSIX 介绍
date: 2020-04-12 20:52:41
tags: Linux
---

>

<!-- more -->


## 一. 场景

##### 1. 程序代码实现同一功能，不同系统内核(操作系统)提供不同的函数调用，例如创建进程，linux下是fork函数，windows下是creatprocess函数。如果在linux下写一个程序，用到fork函数，那么这个程序移植windows要怎么办呢？

##### 2. 通常的做法是把源代码里的fork通通改成creatprocess，然后再重新编译程序，这个方法虽然能解决问题，但是非常不方便。

##### 3. posix标准的出现就是为了解决这个问题。linux和windows都要实现基本的posix标准，linux把fork函数封装成posix_fork，windows把creatprocess函数也封装成posix_fork，都声明在unistd.h里。这样，程序员编写普通应用时候，只用包含unistd.h，调用posix_fork函数，程序就在源代码级别即可在不同操作系统之间移植了。


## 二. 定义
POSIX 表示可移植操作系统接口（Portable Operating System Interface of UNIX，缩写为 POSIX ），操作系统实现了 POSIX接口，应用程序即可通过这些接口，编写跨平台(Linux，Window)的代码。

