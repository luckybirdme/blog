---
title: Linux 查看进程
date: 2020-04-18 15:13:20
tags: Linux
---
> process

<!-- more -->


- 根据关键字查看进程

```sh
[root@VM_133_231_centos ~]# ps -ef |grep nginx
nginx    10308 30681  0  2019 ?        00:10:33 nginx: worker process
root     13932  7384  0 15:13 pts/0    00:00:00 grep --color=auto nginx
root     30681     1  0  2019 ?        00:00:00 nginx: master process /usr/sbin/nginx
```

- 显示进程使用的CPU，内存等信息

```sh
[root@VM_133_231_centos ~]# ps aux | head -n 10
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.2 190644  2108 ?        Ss    2019  53:27 /usr/lib/systemd/systemd --switched-root --system --deserialize 21

```

- 查看僵尸进程，关于[僵尸进程](https://www.jb51.net/article/135562.htm)

```sh
[root@TENCENT64 ~]# ps -ef|grep defunct
root     16515 29700  0  2019 ?        00:00:00 [curl] <defunct>
```

- 显示进程树

```sh
[root@VM_133_231_centos ~]# ps axjf 
 PPID   PID  PGID   SID TTY      TPGID STAT   UID   TIME COMMAND
 # ······
    1   511   511   511 ?           -1 Ss       0  19:40 /usr/sbin/sshd
  511  7382  7382  7382 ?           -1 Ss       0   0:00  \_ sshd: root@pts/0
 7382  7384  7384  7384 pts/0    14567 Ss       0   0:00  |   \_ -bash
 7384 14567 14567  7384 pts/0    14567 R+       0   0:00  |       \_ ps axjf
```

- 查看进程信号

```sh
[root@VM_133_231_centos ~]# kill -l    
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
```
- 信号解释

信号量值	|名称	|说明
-|-|-
1| SIGHUP | 进程挂起
2|	SIGINT	|ctrl +c, 终止信号
3|	SIGQUIT	|ctrl  + \\, 暂停信号，放入后台
4| SIGILL|		非法指令
5|	SIGTRAP	| 	进程异常终止
8| SIGFPE |浮点运算溢出
9|	SIGKILL|	kill - 9 pid, 杀死进程
10| SIGBUS| 总线错误
13|	SIGPIPE		|管道破裂
14|SIGALRM	|	闹钟
15|	SIGTERM	|	缺省终止某个进程，终止掉
17|	SIGSTOP	|	子进程死的时候会给父进程发送这个信号
19|	SIGCONT	|	进程暂停
23|	SIGURG	|	紧急数据
29|	SIGINFO	|	异步 IO


- 强制杀死指定PID的进程，只有一个进程

```sh
kill -9  $PID 
```

- 根据进程名杀死进程，包括多个进程

```sh
killall -9 httpd
```

- 查看使用磁盘的进程

```sh
[root@VM_133_231_centos ~]# fuser -mv /data | head -n 10
                     USER        PID ACCESS COMMAND
/data:               www       F.c.. php-fpm
                     www       F.c.. php-fpm
```


- 查看当前机器CPU和内存使用情况

```sh
top - 15:04:32 up 627 days,  3:10,  4 users,  load average: 0.17, 0.25, 0.29
Tasks: 373 total,   1 running, 371 sleeping,   0 stopped,   1 zombie
%Cpu(s):  0.8 us,  0.2 sy,  0.0 ni, 98.7 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16127020 total,   342132 free,  2272704 used, 13512184 buff/cache
KiB Swap:  2088956 total,  1784972 free,   303984 used. 12732516 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                 
15450 root      20   0   73456  53120   1264 S   4.0  0.3   7123:14 sap1005                                 
  801 root      20   0  299504  16636   1304 S   2.0  0.1 264:34.40 sap1012                                 
 5911 root      20   0    7844   6840    804 S   0.7  0.0   1007:04 sap1002                                 
29700 root      20   0 1426752  60336   4216 S   0.7  0.4   2206:41 python3.7 
```


#### 第一行(top)：
- 目前的时间，亦即是 00:53:59 那个项目；
- 开机到目前为止所经过的时间，亦即是 up 6:07, 那个项目；
- 已经登入系统的用户人数，亦即是 3 users, 项目；
- 系统在 1, 5, 15 分钟的平均工作负载， 越小代表系统越闲置，若高于 1 得要注意你的系统进程是否太过繁复了！

#### 第二行(Tasks)：
- 显示的是目前进程的总量与个别进程在什么状态：显示的是目前进程的总量与个别进程在什么状态(running, sleeping, stopped, zombie)， zombie 是僵尸进程数量。

#### 第三行(Cpus)：
- us(user CPU time)：用户态使用的cpu时间比
- sy(system CPU time)：系统态使用的cpu时间比
- ni(nice CPU time)：用做nice加权的进程分配的用户态cpu时间比
- id(idle)：空闲的cpu时间比
- wa(iowait)：cpu等待磁盘写入完成时间，如果wa特别高，可能是磁盘IO出现问题，使用 iostat，iostop 等分析。
- hi(hardware irq)：硬中断消耗时间
- si(software irq)：软中断消耗时间
- st(steal time)：虚拟机偷取时间
以上数值累加起来等于 CPU 的所有时间片总和 100


#### 第四行(Mem)：
- 表示物理内存使用情况，

#### 第五行(Swap)：
- 表示目前虚拟内存使用情况，如果 swap 被用的很大量，表示系统的物理内存实在不足！


#### 详细进程信息
- PID ：每个 process 的 ID 啦！
- USER ：该 process 所属的使用者；
- PR Priority 的简写，进程的优先执行顺序，越小越早被执行；
- NI Nice 的简写，与 Priority 有关，也是越小越早被执行；
- %CPU CPU 的使用率；
- %MEM ：内存的使用率
- TIME+ CPU 使用时间的累加；


- 在 top 命令下，查看所有 CPU 使用情况，请按 1

```sh
top - 15:25:03 up 627 days,  3:31,  4 users,  load average: 0.95, 0.57, 0.44
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  8.9 us,  3.4 sy,  0.0 ni, 87.6 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16127020 total,   234424 free,  2354516 used, 13538080 buff/cache
KiB Swap:  2088956 total,  1842496 free,   246460 used. 12651008 avail Mem 
top - 15:45:43 up 627 days,  3:52,  4 users,  load average: 0.41, 0.48, 0.48
Tasks: 372 total,   3 running, 369 sleeping,   0 stopped,   0 zombie
%Cpu0  : 14.0 us, 17.8 sy,  0.0 ni, 68.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  : 11.7 us, 18.0 sy,  0.0 ni, 69.4 id,  0.9 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  : 13.8 us, 23.9 sy,  0.0 ni, 62.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  : 10.8 us, 22.5 sy,  0.0 ni, 66.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu4  :  7.8 us, 13.6 sy,  0.0 ni, 72.8 id,  5.8 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu5  :  4.0 us, 15.8 sy,  0.0 ni, 80.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu6  :  5.6 us, 18.5 sy,  0.0 ni, 75.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu7  :  4.8 us, 20.0 sy,  0.0 ni, 75.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

- 在 top 命令下，查看按照 CPU 使用排序，请按大写 P

```sh
top - 15:55:37 up 627 days,  4:01,  4 users,  load average: 0.30, 0.39, 0.43
Tasks: 368 total,   1 running, 367 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.2 sy,  0.0 ni, 99.0 id,  0.2 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16127020 total,   316712 free,  2247236 used, 13563072 buff/cache
KiB Swap:  2088956 total,  1842496 free,   246460 used. 12758656 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                 
15450 root      20   0   73456  53120   1264 S   5.0  0.3   7125:36 sap1005                                 
 5914 root      20   0   24608  11488   1140 S   1.3  0.1 507:44.78 sap1004                                 
30658 root      20   0  332300  43532   5832 S   1.0  0.3   0:20.88 python3.7                               
28470 root      20   0 5570588 259320  12780 S   0.7  1.6  80:10.40 java                                    
```

- 在 top 命令下，查看按照 MEN 使用排序，请按大写 M

```sh
top - 15:56:48 up 627 days,  4:03,  4 users,  load average: 0.57, 0.44, 0.44
Tasks: 367 total,   1 running, 366 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.8 us,  0.1 sy,  0.0 ni, 99.0 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16127020 total,   320608 free,  2246400 used, 13560012 buff/cache
KiB Swap:  2088956 total,  1842496 free,   246460 used. 12759512 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                 
 4546 logsvr    20   0 5892212 385716   7500 S   0.0  2.4 114:39.56 java                                    
28470 root      20   0 5570588 259320  12780 S   0.0  1.6  80:10.54 java                                    
15450 root      20   0   73456  53120   1264 S   5.0  0.3   7125:39 sap1005                                 
 5927 root      20   0   72772  52032   2352 S   0.0  0.3 218:39.77 sap1010                                 
```


- iostat 分析磁盘IO的整体情况

```sh
# 每隔1S输出磁盘IO的详细详细，总共采样 60 次
[root@VM_133_231_centos ~]# iostat -x -k -d 1 60
Linux 3.10.0-514.6.1.el7.x86_64 (VM_133_231_centos) 	04/25/20 	_x86_64_	(1 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda             242.32    34.84   69.16   67.12  2115.31   409.59    37.05     0.09    0.68    1.51    1.55   0.16   2.21

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda              18.18     1.01 2513.13 1852.53 165256.57 157244.44   147.74    73.71   17.09    6.66   31.24   0.18  78.08

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda              16.00     1.00 1484.00 1388.00 140120.00 139284.00   194.57    70.97   24.41   12.68   36.95   0.27  78.10
```

- iotop 查看磁盘 IO 使用较多的进程

```sh
[root@VM_133_231_centos ~]# iotop
Total DISK READ :     110.07 M/s | Total DISK WRITE :     108.06 M/s
Actual DISK READ:     110.07 M/s | Actual DISK WRITE:     109.97 M/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                                                              
10702 be/4 root      110.06 M/s  108.05 M/s  0.00 % 31.57 % cp -i one_g_size.text one_g_size_copy.text
21336 be/4 root       19.68 K/s    0.00 B/s  0.00 %  0.23 % YDService
32765 be/4 root        0.00 B/s    3.94 K/s  0.00 %  0.00 % barad_agent
  512 be/4 polkitd     0.00 B/s    0.00 B/s  0.00 %  0.00 % polkitd --no-debug [gmain]

```
