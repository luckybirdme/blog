---
title: select poll epoll
date: 2020-05-30 22:40:32
tags: Linux
---

> I/O多路复用(I/O multiplexing)

<!-- more -->


## 一. 磁盘读写

##### 1. 阻塞读(blocking)
![](/img/2020/select_poll_epoll_1.png)

当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据（对于网络IO来说，很多时候数据在一开始还没有到达。比如，还没有收到一个完整的UDP包。这个时候kernel就要等待足够的数据到来）。这个过程需要等待，也就是说数据被拷贝到操作系统内核的缓冲区中是需要一个过程的。而在用户进程这边，整个进程会被阻塞（当然，是进程自己选择的阻塞）。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。

所以，blocking IO的特点就是在IO执行的两个阶段都被block了。

##### 2. 非阻塞读(non-blocking)

![](/img/2020/select_poll_epoll_2.png)

当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

所以，nonblocking IO的特点是用户进程需要不断的主动询问kernel数据好了没有


##### 3. 遇到问题

- 无论是阻塞读还是非阻塞读，应用进程都存在阻塞等待，所以严格以上说阻塞和非阻塞读都属于同步的 IO 操作，会导致进程卡住。
- 非阻塞IO需要用户一直轮询操作，轮询可能会来带CPU的占用问题
- 由于同步 IO 导致进程卡住，所以单进程下无法进行多个读 IO 的操作，此时就有IO多路复用(I/O multiplexing)的概念，就是单个进程下操作多个 IO。
- 异步 IO 的定义是应用进程只需发起读取请求，然后就可以执行其他操作了，当数据准备好了并且从内核态拷贝到用户态了，才通知来应用进程处理，所以跟这里的非阻塞 IO 不是同一个概念，异步 IO 需要通过特殊机制来实现，比如 nodejs 的异步 IO 。

![](/img/2020/select_poll_epoll_3.png)



## 二. I/O多路复用

##### 1. 一个进程多个IO操作
- 比如 NGINX，当有多个用户请求不同的静态文件，NGINX 向内核态发起多个文件读取的IO操作，因为每个文件大小不一样，所以文件读取完成的时间也不一样
- 当有某一个文件读取完成，内核态会拷贝数据到用户态并通知 NGINX 处理，NGINX 再根据不同文件，返回给不同的用户。
- 此时的一个 NGINX 进程需要维护多个 IO 操作，区分不同的用户请求，以及内核态返回的IO数据。

![](/img/2020/select_poll_epoll_4.png)

##### 2. 实现原理
- 当用户进程调用了select，那么整个进程会被block，不断的轮询所负责的所有socket。
- 而同时，kernel会监视所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。
- 这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程，此过程也会 block。

![](/img/2020/select_poll_epoll_5.png)

所以，I/O 多路复用的特点是通过一种机制一个进程能同时等待多个文件读取IO，而这些文件IO其中的任意一个进入读就绪状态，select()函数就可以返回。

##### 3. 多个IO操作的要点
- 磁盘读取的IO操作需要通过内核态才能完成，所以存在从用户态切换到内核态的过程。
- 内核态完成文件读取，需要通知用户态处理，因此内核态需要知道哪些IO是哪个用户态进程监听的，所以用户态必须将自己想监听的IO告诉内核态。
- 当用户态收到内核态通知后，需要知道哪些IO收到数据了，不然就只能遍历所有监听的IO，判断哪个IO有数据。




## 三. select

##### 1. 实现原理

```c
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

- select I/O复用模型的关键点在于 fd_set，这是一个整形数组，数组的每个成员的每一位都表示一个文件描述符。
- 在调用select的时候，用户空间将所有要检测的各种事件的文件描述符以fd_set 整型数组的形式传递给内核空间。 
- 内核空间通过轮询监听的描述符集判断是否有监听事件发生。
- 如果某个描述符上有对应的监听事件发生，则该描述符对应的位置1， 跳出阻塞，执行select函数之后的代码；
- 否则，select继续阻塞，CPU资源让出给其他进程。
- 这里牵涉到两次copy，一是从用户态拷贝到内核态，内容为所有监听的文件描述符资源，一是从内核态拷贝到用户态，内容为有事件发生的文件描述符资源。


##### 2. 存在问题
- 每次IO请求调用select，都需要把所有监听的IO集合从用户态拷贝到内核态，这个开销在IO很多时会很大。
- 监听IO的集合采用数组形式保存，为了避免拷贝的性能损耗，32位系统默认最大是 1024，64位默认 2048。
- 当某个IO有返回数据时，需要轮询所有的监听IO描述符，效率较低，如果IO越多开销越大。


## 四. poll

##### 1. 实现原理
- 跟 select 类似的实现方式，只是将数组形式的 fd_set，改造成链表形式，这样就不存在 IO 数量的限制了。

##### 2. 存在问题
- 除了 IO 数量限制没有了，其他缺点跟 select 一样。


## 五. epoll

##### 1. 实现原理

```c
// 创建一个epoll，用于保存监听的IO集合，此句柄存储在用户态和内核态共享的内存地址，因此避免了IO集合的复制
int epoll_create(int size);
// 管理 epoll 的IO集合，包括增删改查一个IO连接，采用红黑树的结构，所以在大量IO连接下，依然保持 log(n) 的复杂度
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
// 等待IO事件返回，返回的IO句柄只是有数据的，而不是整个监听IO集合。
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

(1) int epoll_create(int size);
创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大，这个参数不同于select()中的第一个参数，给出最大监听的fd+1的值，参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议。
当创建好epoll句柄后，它就会占用一个fd值，在linux下如果查看/proc/进程id/fd/，是能够看到这个fd的，所以在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

(2) int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
函数是对指定描述符fd执行op操作。
- epfd：是epoll_create()的返回值。
- op：表示op操作，用三个宏来表示：添加EPOLL_CTL_ADD，删除EPOLL_CTL_DEL，修改EPOLL_CTL_MOD。分别添加、删除和修改对fd的监听事件。
- fd：是需要监听的fd（文件描述符）
- epoll_event：是告诉内核需要监听什么事

(3) int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
等待epfd上的io事件，最多返回maxevents个事件。
参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。


##### 2. 高效原因

- 在调用epoll_create时，在内核cache里建了个红黑树 rbr，用于存储epoll_ctl添加的socket集合，避免了从用户态拷贝到内核态，而且在大量socket下，增删改查的复杂度依然保持在 log(n)
- 当epoll_wait调用时，内核只返回具有数据的socket集合，并采用链表形式保存，而不是所有监听的socket，避免了遍历所有socket才能得知哪个IO有数据，而且链表的增删改操作复杂度接近 O(1)
- epoll 实现方式避免了 select/poll 的三个缺点，没有IO数量的限制，无需从用户态到内核态大量复制，也无需遍历所有IO才知道哪个有数据。


![](/img/2020/select_poll_epoll_7.png)


##### 3. LT(Level Triggered 水平触发)(默认模式) 
当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。

##### 4. ET(Edge Triggered 边沿触发)
当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。
