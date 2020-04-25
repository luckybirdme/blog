---
title: Linux /dev/shm
date: 2020-04-24 10:54:59
tags: Linux
---

> /dev/shm, shared memory

<!-- more -->

## 一. 由来
1. Linux 利用内存虚拟出来的目录，目录中的文件都是保存在内存中，而不是磁盘上。
2. 目录其大小是非固定的，即不是预先分配好的内存，默认是系统内存的一半。

```sh
[root@VM_133_231_centos shm]# free -h
              total        used        free      shared  buff/cache   available
Mem:           992M        601M         65M        12M        326M         83M
Swap:          1.0G        489M        534M
# /dev/shm 占用了 497M, 是总内存 992M 的一半
[root@VM_133_231_centos shm]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  9.7G  8.9G  53% /
devtmpfs        487M     0  487M   0% /dev
tmpfs           497M   48K  497M   1% /dev/shm
```

## 二. 优点
1. tmpfs 是一个文件系统，而不是块设备，只是安装它，就可以使用了。
2. 因为 /dev/shm 目录内容是保存在内存中的，所以读写特别快。
3. 可以动态调整 /dev/shm 目录的大小，以便满足需求。

```sh
[root@VM_133_231_centos shm]# dd if=/dev/zero of=/dev/shm/123.random bs=1M count=100 
100+0 records in
100+0 records out
104857600 bytes (105 MB) copied, 0.0736356 s, 1.4 GB/s
# /dev/shm 使用了 100M
[root@VM_133_231_centos shm]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  9.7G  8.9G  53% /
devtmpfs        487M     0  487M   0% /dev
tmpfs           497M  101M  397M  21% /dev/shm
# shared 显示使用了 100M
[root@VM_133_231_centos shm]# free -h
              total        used        free      shared  buff/cache   available
Mem:           992M        601M         65M        112M        326M         83M
Swap:          1.0G        489M        534M
# 更改 /dev/shm 目录大小为 200M
[root@VM_133_231_centos shm]# mount -o size=200M  -o remount /dev/shm
[root@VM_133_231_centos shm]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  9.7G  8.9G  53% /
devtmpfs        487M     0  487M   0% /dev
tmpfs           200M  101M  100M  51% /dev/shm
```

## 三. 缺点
1. tmpfs 数据在重新启动之后不会保留，因为虚拟内存本质上就是易失的，重启后需要手动恢复数据。

