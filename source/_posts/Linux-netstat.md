---
title: Linux netstat
date: 2020-04-18 16:50:52
tags: Linux
---
> netstat

<!-- more -->

## 一. 简介
Netstat 命令用于显示各种网络相关信息，如网络连接，路由表，接口状态 (Interface Statistics)，masquerade 连接，多播成员 (Multicast Memberships) 等等。

## 二. 参数
参数| 作用
-|-
-a |(all)显示所有选项，netstat默认不显示LISTEN相关
-t |(tcp)仅显示tcp相关选项
-u |(udp)仅显示udp相关选项
-n |拒绝显示别名，能显示数字的全部转化成数字。(重要)
-l |仅列出有在 Listen (监听) 的服務状态
-p |显示建立相关链接的程序名(macOS中表示协议 -p protocol)
-r |显示路由信息，路由表
-e |显示扩展信息，例如uid等
-s |按各个协议进行统计 (重要)
-c |每隔一个固定时间，执行该netstat命令。


## 三. 使用

- 列出所有端口
```sh
netstat -a 
```

- 列出所有的TCP端口
```sh
netstat -at 
```

- 列出所有的UDP端口
```sh
netstat -au 
```

- 列出所有处于监听状态的socket
```sh
netstat -l 
```

- 列出所有监听TCP端口的socket
```sh
netstat -lt 
```

- 列出所有监听UDP端口的socket
```sh
netstat -lu 
```

- 查看本机路由表
```sh
netstat -rn
```

- 找出程序运行的端口
```sh
netstat -ap | grep nginx
```

- 找出运行在指定端口的进程
```sh
netstat -an | grep ':80'
```
