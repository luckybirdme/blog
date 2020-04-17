---
title: Linux 文件分类
date: 2020-04-12 23:07:12
tags: Linux
---

> 文件分类

<!-- more -->


- [-] 正规文件，包括文本，压缩包，二进制文件等
```sh
[root@VM_133_231_centos dev]# ll /root | grep ^-
-rw-r--r--   1 root     root          2920 Jul  1  2018 10558.txt
-rw-r--r--   1 root     root        253011 Aug 29  2016 PyYAML-3.12.tar.gz
```
- [d] 目录
```sh
[root@VM_133_231_centos dev]# ll / | grep ^d
dr-xr-xr-x.   4 root root       4096 May 18  2018 boot
drwxr-xr-x   18 root root       2880 Feb 20  2019 dev
```
- [l] 软连接
```sh
[root@VM_133_231_centos dev]# ll /dev/ | grep ^l
lrwxrwxrwx 1 root root          11 Feb 20  2019 core -> /proc/kcore
lrwxrwxrwx 1 root root          13 Feb 20  2019 fd -> /proc/self/fd
```
- [p] 管道 pipe
```sh
[root@VM_133_231_centos dev]# ll /var/run/ | grep ^p
prw-------  1 root   root              0 Feb 20  2019 dmeventd-client
prw-------  1 root   root              0 Feb 20  2019 dmeventd-server
```
- [s] socket 接口
```sh
[root@VM_133_231_centos dev]# ll /var/run/ | grep ^s
srw-rw-rw-  1 root   root              0 Feb 20  2019 acpid.socket
srw-rw-rw-  1 root   root              0 Feb 20  2019 rpcbind.sock
```
- [b] 区块设备，比如硬盘，软盘
```sh
[root@VM_133_231_centos dev]# ll /dev/ | grep ^b
brw-rw---- 1 root disk    253,   0 Feb 20  2019 vda
brw-rw---- 1 root disk    253,   1 Feb 20  2019 vda1
```

- [c] 字符串设备，比如键盘、鼠标等
```sh
[root@VM_133_231_centos dev]# ll /dev/ | grep ^c
crw------- 1 root root     10, 235 Feb 20  2019 autofs
crw------- 1 root root      5,   1 Feb 20  2019 console
```