---
title: Linux 硬链接和软连接
date: 2020-04-14 08:15:07
tags: Linux
---

> ln

<!-- more -->


## 一. 创建方法
##### 1. 硬链接
```sh
[root@VM_133_231_centos ~]# ln test_src_file.txt test_hard_link.txt
[root@VM_133_231_centos ~]# ls -il test_*
131224 -rw-r--r-- 2 root root 14 Apr 14 08:09 test_hard_link.txt
131224 -rw-r--r-- 2 root root 14 Apr 14 08:09 test_src_file.txt
```

##### 2. 软连接
```sh
[root@VM_133_231_centos ~]# ln -s test_src_file.txt test_soft_link.txt
[root@VM_133_231_centos ~]# ls -il test_*
131609 lrwxrwxrwx 1 root root 17 Apr 14 08:10 test_soft_link.txt -> test_src_file.txt
131224 -rw-r--r-- 2 root root 14 Apr 14 08:09 test_src_file.txt
```

## 二. 两者区别
##### 1. 硬链接
- 指向同一个 inode, 同一种文件形式存在, 创建时间跟源文件一样，而且两个文件会同步更新，实际上文件内容保存在同一个 block 中。
- 相当于 cp -p 命令复制文件，外加同步更新的机制。如果删除了源文件，硬链接的文件依然存在，可以正常使用。
- 因为使用了相同的 inode 和 block, 所以硬链接和源文件必须在同一个 filesystem 。
- 不支持目录建立硬链接

##### 2. 软连接
- 创建另外一种文件形式 l, inode 已经不相同，创建时间跟源文件也不一样，但是两个文件会同步更新。
- 相当于给源文件创建了一个 window 的快捷方式。如果删除了源文件，软连接的指向就为空，也就无法正常使用了。
- 支持在不同的 filesystem 之间建立软连接。
- 支持目录建立软连接。
