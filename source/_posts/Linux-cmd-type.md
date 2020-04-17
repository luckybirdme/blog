---
title: Linux 内建命令
date: 2020-04-15 08:12:42
tags: Linux
---

> shell

<!-- more -->


## 一. 内建命令
实际上是shell程序的一部分，其中包含的是一些比较简单的linux系统命令，这些命令由shell程序识别并在shell程序内部完成运行，通常在linux系统加载运行时shell就被加载并驻留在系统内存中。

内建命令是写在bash源码里面的，其执行速度比外部命令快，因为解析内部命令shell不需要创建子进程。比如：exit，history，cd，echo等。


## 二. 外部命令
是linux系统中的应用程序部分，因为应用程序的功能通常都比较强大，所以其包含的程序量也会很大，在系统加载时并不随系统一起被加载到内存中，而是在需要时才将其调用内存。

通常外部命令的实体并不包含在shell中，但是其命令执行过程是由shell程序控制的。shell程序管理外部命令执行的路径查找、加载，并 fork 新进程来执行命令，所以执行起来相对shell内建命令会较慢。

外部命令是在bash之外额外安装的，通常放在/bin，/sbin，/usr/bin，/usr/sbin，/usr/local/sbin等等。可通过“echo $PATH”命令查看外部命令的存储路径，比如：ls、vim等。

## 三. type 区分内外部命令

```sh
# 内建命令
[root@VM_133_231_centos ~]# type cd
cd is a shell builtin
[root@VM_133_231_centos ~]# type history 
history is a shell builtin
# 外部命令
[root@VM_133_231_centos ~]# type mkdir
mkdir is /usr/bin/mkdir
[root@VM_133_231_centos ~]# type touch   
touch is /usr/bin/touch
[root@VM_133_231_centos ~]# 

```