---
title: Linux lsof
date: 2020-04-18 16:50:44
tags: Linux
---
> list open file

<!-- more -->

## 一. 简介
在 linux 系统中，一切皆文件。通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。所以 lsof 命令不仅可以查看进程打开的文件、目录，还可以查看进程监听的端口等 socket 相关的信息。

## 二. 参数

参数|作用
-|-
-a |指示其它选项之间为与的关系
-c |<进程名> 输出指定进程所打开的文件
-d |<文件描述符> 列出占用该文件号的进程
+d |<目录>  输出目录及目录下被打开的文件和目录(不递归)
+D| <目录>  递归输出及目录下被打开的文件和目录
-i |<条件>  输出符合条件与网络相关的文件
-n |不解析主机名
-p |<进程号> 输出指定 PID 的进程所打开的文件
-P |不解析端口号
-t |只输出 PID
-u |输出指定用户打开的文件
-U |输出打开的 UNIX domain socket 文件
-h |显示帮助信息
-v |显示版本信息

## 三. 使用

- 查看哪些进程打开了某个文件
```sh
[root@VM_133_231_centos ~]# lsof /var/log/messages
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF  NODE NAME
rsyslogd 460 root    5w   REG  253,1 37468135 25858 /var/log/messages
```

- 查看哪些进程使用了当前目录
```sh
[root@VM_133_231_centos ~]# lsof +d /var/log         
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF  NODE NAME
rsyslogd 460 root    3w   REG  253,1 20482903 24835 /var/log/cron
rsyslogd 460 root    4w   REG  253,1  3275751 24828 /var/log/secure
rsyslogd 460 root    5w   REG  253,1 37468135 25858 /var/log/messages

# 递归查找
[root@VM_133_231_centos ~]# lsof +D /usr/local/
COMMAND     PID   USER   FD   TYPE             DEVICE SIZE/OFF      NODE NAME
php56-fpm  1093   root  cwd    DIR              253,1     4096    150135 /usr/local/php56/sbin
php56-fpm  1093   root  txt    REG              253,1 35280344    150139 /usr/local/php56/sbin/php-fpm
sgagent    1931   root  cwd    DIR              253,1     4096    222398 /usr/local/qcloud/stargate
sgagent    1931   root  txt    REG              253,1   366568    222429 /usr/local/qcloud/stargate/sgagent64
sgagent    1931   root    4w   REG              253,1  3989169    221433 /usr/local/qcloud/stargate/logs/stargate.log

```


- 查看进程打开了哪些文件
```sh
[root@VM_133_231_centos ~]# ps -ef | grep nginx | grep -v grep
nginx    10308 30681  0  2019 ?        00:10:33 nginx: worker process
root     30681     1  0  2019 ?        00:00:00 nginx: master process /usr/sbin/nginx
[root@VM_133_231_centos ~]# lsof -p 30681
COMMAND   PID USER   FD   TYPE             DEVICE SIZE/OFF      NODE NAME
nginx   30681 root  txt    REG              253,1  1101400    144607 /usr/sbin/nginx
nginx   30681 root  mem    REG              253,1    23976    263872 /usr/lib64/perl5/vendor_perl/auto/nginx/nginx.so
nginx   30681 root  mem    REG              253,1    80440    263878 /usr/lib64/nginx/modules/ngx_stream_module.so

```

- 查看用户打开了哪些文件
```sh
[root@VM_133_231_centos ~]# lsof -u nginx
COMMAND   PID  USER   FD      TYPE             DEVICE SIZE/OFF      NODE NAME
nginx   10308 nginx  txt       REG              253,1  1101400    144607 /usr/sbin/nginx
nginx   10308 nginx  mem       REG              253,1    37352    144283 /usr/lib64/libnss_sss.so.2
nginx   10308 nginx  mem       REG              253,1    23976    263872 /usr/lib64/perl5/vendor_perl/auto/nginx/nginx.so
```

- 查看 TCP 协议的文件
```sh
[root@VM_133_231_centos ~]# lsof -i tcp  
COMMAND     PID   USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
systemd       1   root   45u  IPv4     11299      0t0  TCP *:sunrpc (LISTEN)
sshd        511   root    3u  IPv4     12854      0t0  TCP *:ssh (LISTEN)
sshd        511   root    4u  IPv6     12863      0t0  TCP *:ssh (LISTEN)
mongod      674 mongod    8u  IPv4     13382      0t0  TCP localhost:27017 (LISTEN)
php56-fpm  1093   root    7u  IPv4   2020971      0t0  TCP localhost:etlservicemgr (LISTEN)

```

- 查看指定端口的文件
```sh

[root@VM_133_231_centos ~]# lsof -i:80
COMMAND   PID  USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
nginx   10308 nginx   20u  IPv4 2011425      0t0  TCP *:http (LISTEN)
nginx   10308 nginx   21u  IPv6 2011426      0t0  TCP *:http (LISTEN)
nginx   30681  root   20u  IPv4 2011425      0t0  TCP *:http (LISTEN)
nginx   30681  root   21u  IPv6 2011426      0t0  TCP *:http (LISTEN)

```

- 查看与指定host和port关联的文件
```sh
lsof -i @other.host.com:80
```

- 将指定用户打开的文件列表信息，同时是TCP网络连接信息的一起输出
```sh
lsof -a -u nginx -i tcp
```