---
title: Linux 安装软件
date: 2020-04-19 08:47:48
tags: Linux
---

> 本文基于 Centos 7

<!-- more -->

## 一. 原理
1. Linux 系统可执行是二进制文件，即以 01 为内容的 binary file
2. 开发人员编写应用程序的一般是纯文本文件，也叫 ASCII 文件，比如 C 语言的 printf("Hello World")

```sh
# ELF 64-bit LSB executable, 二进制文件
[root@VM_133_231_centos ~]# file /usr/sbin/nginx
/usr/sbin/nginx: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=5966c969375b59d5236e9dc22d8b282bd57967a2, stripped
#  ASCII text executable, 纯文本文件, shell 语言最终也会转化成二进制文件来执行
[root@VM_133_231_centos ~]# file /etc/init.d/network 
/etc/init.d/network: Bourne-Again shell script, ASCII text executable
```

3. 纯文本文件想在 Linux 系统执行，则必须经过编译成二进制文件，以 C 语言为例，借助 gcc 编译才能在 Linux 上执行

```sh

[root@VM_133_231_centos test_c]# cat hello.c 
#include <stdio.h> 
int main(void) { 
	printf("Hello World\n"); 
}
[root@VM_133_231_centos test_c]# gcc hello.c 
[root@VM_133_231_centos test_c]# ls -rhtl
total 16K
-rw-r--r-- 1 root root   67 Apr 19 10:34 hello.c
-rwxr-xr-x 1 root root 8.4K Apr 19 10:34 a.out
[root@VM_133_231_centos test_c]# ./a.out 
Hello World

```

![](/img/2020/Linux_run_binary.png)


4. 应用程序除了本身业务逻辑，也会引用操作系统的类库，即函数库，比如 C 语言 include 的文件 
5. 由于不同操作系统以及版本，函数库肯定也会不一样，因此编译应用程序之前，必须先检查当前操作系统的函数库
6. 应用程序的安装包一般会 configure 文件，执行该文件就是进行函数库的检查，各种 check for 是为了确保当前系统支持该应用程序。

```sh
[root@VM_133_231_centos php-5.6.40]# ./configure 
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking for a sed that does not truncate output... /usr/bin/sed
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking target system type... x86_64-unknown-linux-gnu
```

7. 应用程序包括多个文件，手动执行每个文件编译会比较麻烦，只更新一个文件时希望增量编译，这时候就需要定义个编译规则，这个规则通过 Makefile 文件来定义，一般在执行 configure 后就会生成 Makefile 文件

```sh
[root@VM_133_231_centos php-5.6.40]# ls -l Makefile*
-rw-r--r-- 1 root     root     117027 Apr 19 09:44 Makefile
-rw-r--r-- 1 www-data www-data   1115 Jan  9  2019 Makefile.frag
-rw-r--r-- 1 root     root      10366 Apr 19 09:44 Makefile.fragments
-rw-r--r-- 1 www-data www-data   2485 Jan  9  2019 Makefile.gcov
-rw-r--r-- 1 www-data www-data   6760 Jan  9  2019 Makefile.global
-rw-r--r-- 1 root     root      87172 Apr 19 09:44 Makefile.objects
```


8. 当生成 Makefile 后，通过 make 命令对应用程序的源代码进行编译，最终会输出 Linux 可执行的二进制文件。
9. 为了将应用程序的可执行二进制文件链接到系统路径 PATH，最后再执行 make install，即可任务地方执行了。
```sh
[root@VM_133_231_centos bin]# ls -rhtl /usr/bin/php56 
lrwxrwxrwx 1 root root 24 Feb 23  2019 /usr/bin/php56 -> /usr/local/php56/bin/php
[root@VM_133_231_centos bin]# php56 -v
PHP 5.6.40 (cli) (built: Feb 23 2019 17:05:04) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies

```

## 二. RPM

1. 遇到问题
- 利用源代码编译安装软件，如果某个步骤异常，会导致安装失败
- 某些软件之间可能会相互依赖，想要安装 A ，必须先安装 B

2. RedHat 公司推出 RPM(RedHat Package Manager) 软件包管理工具，解决上述问题
- 将已经编译完成的软件二进制文件打包，传输与安装上都很方便，不需要再重新编译。
- 软件之间依赖信息和安装记录都保存在 Linux 主机的数据库上，方便查询、安装与升级。
- 如果 Linux 主机本地没有对应的依赖安装包，通过 yum 命令到远程软件仓库下载并安装。

3. RPM 命名分类

```sh
[root@VM_133_231_centos ~]# rpm -qa | grep php
php-symfony-common-2.8.12-2.el7.noarch
# 软件名称 php
# 版本好 5.4.14
# 打包次数 46
# 硬件平台 el7.x86_64
php-5.4.16-46.el7.x86_64
```

硬件平台编码|解释
-|-
i386|几乎适用于所有的 x86 平台(32位)
x86_64|针对 64 位的 CPU 进行优化编译设定
noarch|没有任何硬件等级上的限制。一般没有 binary program 存在，多属于 shell script 编写的软件。

4. RPM 常用指令

```sh
# 根据关键字查找安装包
[root@VM_133_231_centos ~]# rpm -qa | grep nginx
nginx-all-modules-1.10.2-1.el7.noarch
nginx-1.10.2-1.el7.x86_64
nginx-filesystem-1.10.2-1.el7.noarch
······
# 查看具体安装包的信息
[root@VM_133_231_centos ~]# rpm -qi nginx-1.10.2-1.el7.x86_64
Name        : nginx
Epoch       : 1
Version     : 1.10.2
Release     : 1.el7
Architecture: x86_64
Install Date: Sat Feb 18 16:16:32 2017
Group       : System Environment/Daemons
Size        : 1459372
License     : BSD
Signature   : RSA/SHA256, Fri Nov  4 16:25:30 2016, Key ID 6a2faea2352c64e5
Source RPM  : nginx-1.10.2-1.el7.src.rpm
Build Date  : Mon Oct 31 20:39:41 2016
······
# 查看安装包相关目录
[root@VM_133_231_centos ~]# rpm -ql nginx
/etc/logrotate.d/nginx
/etc/nginx/fastcgi.conf
/etc/nginx/fastcgi.conf.default
/etc/nginx/fastcgi_params
/etc/nginx/fastcgi_params.default
/etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/mime.types
/etc/nginx/mime.types.default
/etc/nginx/nginx.conf
······

# 安装原版光盘上的软件 
[root@VM_133_231_centos ~]# rpm -ivh nginx-1.10.2-1.el7.src.rpm
# 安装多个
[root@VM_133_231_centos ~]# rpm -ivh a.rpm b.rpm *.rpm 
# 直接由网络上面的某个文件安装
[root@VM_133_231_centos ~]# rpm -ivh http://website.name/path/pkgname.rpm

```


## 三. yum

1. 对于Linux 主机本地没有 rpm 安装包的软件，可通过 yum 命令到远程软件仓库下载并安装

2. yum 常用命令
```sh
# 查看软件信息，包括可用和已经安装的包
[root@VM_133_231_centos ~]# yum info redis
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
Available Packages
Name        : redis
Arch        : x86_64
Version     : 3.2.12
Release     : 2.el7
Size        : 544 k
Repo        : epel/7/x86_64
Summary     : A persistent key-value database
URL         : http://redis.io
······
[root@VM_133_231_centos ~]# yum list nginx
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
Installed Packages
nginx.x86_64                                                          1:1.10.2-1.el7                                                           @epel
Available Packages
nginx.x86_64                                                          1:1.16.1-1.el7                                                           epel 

# 安装对应包，需要用户输入 y 来确认
[root@VM_133_231_centos ~]# yum install redis
# 自动输入 y，即自动安装
[root@VM_133_231_centos ~]# yum install redis -y
# 卸载安装包
[root@VM_133_231_centos ~]# yum remove redis

```

3. yum 的远程软件仓库，即 yum 源，需要联网才能下载安装包，而不同 yum 源，下载速度也不一样，建议使用国内的 yum 源，比如[阿里云](https://developer.aliyun.com/mirror/centos)，[腾讯云](https://mirrors.cloud.tencent.com/)，设置方法大同小异

```sh
# 备份当前 yum 源，默认是 RedHat
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

# 获取国内 yum 源的配置文件，主要是 baseurl 更换成国内 yum 源地址
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 拉取软件包基本信息，作为缓存使用
yum makecache

# 使用国内 yum 源，跟之前命令一样
yum install pkgname

```