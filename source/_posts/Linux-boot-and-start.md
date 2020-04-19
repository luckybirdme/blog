---
title: Linux 开机过程
date: 2020-04-18 12:32:19
tags: Linux
---

> 本文以 Centos 7 为例

<!-- more -->


## 一. 启动流程图

![](/img/2020/Linux_boot.png)

1. 加载 BIOS 的硬件信息与进行测试，并依据设定取得第一个可开机的装置；
2. 读取并执行第一个磁盘引导区 MBR(Main Boot Record) 的 boot Loader ，亦即是 grub2 程序；
3. 依据 boot loader 的设定加载 Linux 核心 Kernel，再次侦测硬件，并加载驱动程序；
4. 在硬件驱动成功后， Kernel 会主动呼叫 systemd 程序，默认运行级别是多用户模式，即 multi-user.target
- systemd 执行 sysinit.target.wants 初始化操作系统；
- systemd 启动 multi-user.target.wats 下的本机与服务器服务；
- systemd 执行 rc-local.service，即 /etc/rc.d/rc.local
- systemd 执行 getty.target.wants 设置登录终端的信息；
- 如果是运行图形界面模式，systemd 会执行 graphical.target.wants 需要的服务
5. 至此，systemd 执行完毕，Linux 系统进入正常运行阶段。


```sh
# centos 7 通过 systemd 管理的启动进程
[root@VM_133_231_centos ~]# ll /etc/systemd/system
total 28
lrwxrwxrwx. 1 root root   37 Apr 21  2016 default.target -> /lib/systemd/system/multi-user.target
drwxr-xr-x. 2 root root 4096 Apr 21  2016 default.target.wants
drwxr-xr-x. 2 root root 4096 Apr 21  2016 getty.target.wants
drwxr-xr-x. 2 root root 4096 Apr 18 22:55 multi-user.target.wants
-rw-r--r--  1 root root  655 Nov 10  2016 rc-local.service
drwxr-xr-x. 2 root root 4096 Feb 18  2017 sockets.target.wants
drwxr-xr-x. 2 root root 4096 Apr 21  2016 sysinit.target.wants
drwxr-xr-x. 2 root root 4096 Apr 21  2016 system-update.target.wants

# centos 7 以下版本通过 /etc/inittab 设置运行级别
[root@VM_133_231_centos ~]# cat /etc/inittab 
# inittab is no longer used when using systemd.
#
# ADDING CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# Ctrl-Alt-Delete is handled by /usr/lib/systemd/system/ctrl-alt-del.target
#
# systemd uses 'targets' instead of runlevels. By default, there are two main targets:
#
# multi-user.target: analogous to runlevel 3
# graphical.target: analogous to runlevel 5
#
# To view current default target, run:
# systemctl get-default
#
# To set a default target, run:
# systemctl set-default TARGET.target
#
[root@VM_133_231_centos ~]# systemctl get-default
multi-user.target
[root@VM_133_231_centos ~]# runlevel 
N 3
```

运行级别数值 | 表示含义
-|-
0|表示关机
1|单用户模式
2|无网络连接的多用户命令行模式，没有NFS
3|有网络连接的多用户命令行模式，默认模式
4|暂时没有使用
5|带图形界面的多用户模式
6|重新启动



## 二. 开机自启动

-  借助 /etc/rc.local, 将会在开机时执行该文件，此方法比较粗暴
```sh
[root@VM_133_231_centos ~]# cat /etc/rc.local 
#!/bin/sh
cd /usr/local/nginx/bin && ./start.sh
```

- 使用 chkconfig
```sh
#列出所有的系统服务
chkconfig --list        
#增加httpd服务
chkconfig --add httpd        
#设置httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态
chkconfig --level httpd 2345 on        
```


- Centos 7 采用 systemd 管理 init，通过 systemctl 命令进行操作，它是 service 和 chkconfig 命令集合，service 实质是执行 /etc/init.d/ 下的脚本。通过 systemctl 不仅可以设置开机启动进程，还能像 service 命令一样操作进程。

```sh
# service nginx status
[root@ ~]# cat /etc/init.d/nginx.sh 
#! /bin/sh

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
CMD=/usr/local/nginx/sbin/nginx
PID_FILE=/usr/local/nginx/logs/nginx.pid
NAME=nginx
DESC=nginx

test -x $CMD || exit 0

case "$1" in
  start)
        echo -n "Starting $DESC: "
        $CMD
        echo "$NAME."
        ;;
  stop)
        echo -n "Stopping $DESC: "
        /bin/kill -QUIT `cat $PID_FILE`
        echo "$NAME."
        ;;
  reload)
        echo -n "Reloading $DESC configuration: "
         /bin/kill -HUP `cat $PID_FILE`
        echo "$NAME."
        ;;
  *)
        echo "Usage: $NAME {start|stop|reload}" >&2
        exit 1
        ;;
esac

exit 0

# 使用 systemctl
[root@VM_133_231_centos ~]# systemctl status nginx
* nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-02-23 17:22:33 CST; 1 years 1 months ago
 Main PID: 30681 (nginx)
   CGroup: /system.slice/nginx.service
           |-10308 nginx: worker process
           `-30681 nginx: master process /usr/sbin/nginx
# 启停服务
[root@VM_133_231_centos ~]# systemctl stop nginx
[root@VM_133_231_centos ~]# systemctl start nginx


```

- systemctl 设置开机自启动需要编写 /lib/systemd/system/process.service 文件，以 nginx.service 为例子

```sh
# 启停脚本所在目录
[root@VM_133_231_centos ~]# ll /lib/systemd/system | grep nginx
-rw-r--r--  1 root root  618 Oct 31  2016 nginx.service
# 脚本内容
[root@VM_133_231_centos ~]# cat /lib/systemd/system/nginx.service  
[Unit]
Description=The nginx HTTP and reverse proxy server
# 启动顺序
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
# 启动前执行的动作
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
# 启动命令
ExecStart=/usr/sbin/nginx
# reload 命令
ExecReload=/bin/kill -s HUP $MAINPID
# 停止命令
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
# 在多用户模式下安装
WantedBy=multi-user.target

```

- systemctl 设置开机自启动，其实只是通过软连接的方式，把启停脚本放到指定运行模式下的 systemd 的进程目录，nginx 安装在多用户运行模式，所以启停脚本就放到 system/multi-user.target.wants 目录下
```sh
# 查看启停脚本是否有效，以及开机状态
[root@VM_133_231_centos ~]# systemctl list-unit-files | grep nginx
nginx.service                                 disabled
# 设置开机启动
[root@VM_133_231_centos ~]# systemctl enable nginx                
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.
[root@VM_133_231_centos ~]# systemctl list-unit-files | grep nginx
nginx.service                                 enabled 

```

