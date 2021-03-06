---
title: 内核态和用户态
date: 2020-05-30 15:49:07
tags: Linux
---

> 内核态
> 用户态

<!-- more -->


## 一. Linux 体系框架

![](/img/2020/linux_system_user_1.png)

##### 1. 内核
操作系统运行的地方，利用驱动程序跟硬件交互，比如 CPU、磁盘、网卡等

##### 2. 系统调用
为了使上层应用能够访问到底层硬件资源，内核为上层应用提供的访问接口，各个操作系统可能会不一样，不同版本的操作系统也可能不一样。
比如 Linux 函数 open, close, read, write, ioctl等，需包含头文件 unistd.h

##### 3. 公共函数库
将系统调用包装，提供更丰富的应用场景，更方便应用程序调用，以 C 库函数为主流，而所有 C 函数库是在不同操作下是相同的。
比如 C 库函数 system, fprintf, malloc等，fprintf 底层是通过系统调用的 write 函数实现的。

##### 4. shell
外壳的意思，一个特殊的应用程序，俗称命令行，本质上是一个命令解释器，它下通系统调用，上通各种应用。
每个终端用户都对应一个shell交互窗口，Shell也是编程的，标准的语法，符合其语法的文本叫Shell脚本。

##### 5. 应用程序
是用户编写的应用程序，通过系统调用或者公共函数库使用内核的硬件资源，比如 NGINX。

## 二. 内核态(Kernel Mode)和用户态(User Mode)

##### 1. 原因
- 在CPU的所有指令中，有一些指令是非常危险的，如果错用，将导致整个系统崩溃。所以，CPU将指令分为特权指令和非特权指令，对于那些危险的指令，只允许操作系统及其相关模块使用，普通的应用程序只能使用那些不会造成灾难的指令。

- intel cpu 提供Ring0-Ring3四种级别的运行模式，Ring0级别最高，Ring3最低。Linux使用了Ring3级别运行用户态，Ring0作为 内核态。
 

##### 2. 用户态切换到内核态
- 用户程序都是运行在用户态的, 但是需要做一些内核态的事情, 例如从硬盘读取数据, 此时用户程序需要切换到内核态执行这些操作。这方式就是系统调用, 在CPU中的实现称之为陷阱指令(Trap Instruction)。
- 用户态和内核态使用的资源不一样，比如虚拟内存地址，堆栈地址，所以用户态切换到内核态时，会涉及到上下文切换，此过程是对性能有一定的性能损害。


![](/img/2020/linux_system_user_2.png)


##### 3. 切换的主要方法

- 系统调用，也称为软中断，用户态进程通过系统调用申请使用操作系统提供的服务程序完成工作，比如调用 write 方法，而系统调用的核心是使用了操作系统为用户特别开放的一个中断来实现，例如 Linux 的 int 0x80 中断。

- 异常事件，当CPU正在执行运行在用户态的程序时，突然发生某些预先不可知的异常事件，这个时候就会触发从当前用户态执行的进程转向内核态执行相关的异常事件，典型的如缺页异常。

- 外围设备的中断，当外围设备完成用户的请求操作后，会向CPU发出中断信号，此时，CPU就会暂停执行下一条即将要执行的指令，转而去执行中断信号对应的处理程序，如果先前执行的指令是在用户态下，则自然就发生从用户态到内核态的转换。

