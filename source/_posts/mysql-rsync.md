---
title: MySQL 同步机制
date: 2020-07-11 16:12:12
tags: MySQL
---

> 异步、半同步、全同步

<!-- more -->


MySQL主要数据复制类型分别是异步、半同步、全同步

![](/img/2020/mysql_rsync_1.jpeg)

## 一. 异步复制(Asynchronous replication)

1. 逻辑上
MySQL默认的复制即是异步的，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理，这样就会有一个问题，主如果crash掉了，此时主上已经提交的事务可能并没有传到从库上，如果此时，强行将从提升为主，可能导致新主上的数据不完整。

2. 技术上
主库将事务 Binlog 事件写入到 Binlog 文件中，此时主库只会通知一下 Dump 线程发送这些新的 Binlog，然后主库就会继续处理提交操作，而此时不会保证这些 Binlog 传到任何一个从库节点上。

3. 原理图

![](/img/2020/mysql_rsync_2.jpeg)


(1) 在Slave 服务器上执行sart slave命令开启主从复制开关，开始进行主从复制。
(2) 此时，Slave服务器的IO线程会通过在master上已经授权的复制用户权限请求连接master服务器，并请求从执行binlog日志文件的指定位置(日志文件名和位置就是在配置主从复制服务时执行change master命令指定的)之后开始发送binlog日志内容
(3) Master服务器接收到来自Slave服务器的IO线程的请求后，其上负责复制的IO线程会根据Slave服务器的IO线程请求的信息分批读取指定binlog日志文件指定位置之后的binlog日志信息，然后返回给Slave端的IO线程。返回的信息中除了binlog日志内容外，还有在Master服务器端记录的IO线程。返回的信息中除了binlog中的下一个指定更新位置。
(4) 当Slave服务器的IO线程获取到Master服务器上IO线程发送的日志内容、日志文件及位置点后，会将binlog日志内容依次写到Slave端自身的Relay Log(即中继日志)文件(Mysql-relay-bin.xxx)的最末端，并将新的binlog文件名和位置记录到master-info文件中，以便下一次读取master端新binlog日志时能告诉Master服务器从新binlog日志的指定文件及位置开始读取新的binlog日志内容
(5) Slave服务器端的SQL线程会实时检测本地Relay Log 中IO线程新增的日志内容，然后及时把Relay LOG 文件中的内容解析成sql语句，并在自身Slave服务器上按解析SQL语句的位置顺序执行应用这样sql语句，并在relay-log.info中记录当前应用中继日志的文件名和位置点


## 二. 全同步复制(Fully synchronous replication)

1. 逻辑上
指当主库执行完一个事务，所有的从库都执行了该事务才返回给客户端。因为需要等待所有从库执行完该事务才能返回，所以全同步复制的性能必然会收到严重的影响。

2. 技术上
当主库提交事务之后，所有的从库节点必须收到、APPLY并且提交这些事务，然后主库线程才能继续做后续操作。但缺点是，主库完成一个事务的时间会被拉长，性能降低。

3. 原理图

![](/img/2020/mysql_rsync_3.jpeg)


## 三. 半同步复制(Semisynchronous replication)

1. 逻辑上
是介于全同步复制与全异步复制之间的一种，主库只需要等待至少一个从库节点收到并且 Flush Binlog 到 Relay Log 文件即可，主库不需要等待所有从库给主库反馈。同时，这里只是一个收到的反馈，而不是已经完全完成并且提交的反馈，如此，节省了很多时间。

2. 技术上
介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到并写到relay log中才返回给客户端。相对于异步复制，半同步复制提高了数据的安全性，同时它也造成了一定程度的延迟，这个延迟最少是一个TCP/IP往返的时间。所以，半同步复制最好在低延时的网络中使用。

3. 原理图
master将每个事务写入binlog(sync_binlog=1)，传递到slave刷新到磁盘(sync_relay=1)，同时主库提交事务(commit)。master等待slave反馈收到relay log，只有收到ACK后master才将commit OK结果反馈给客户端。

![](/img/2020/mysql_rsync_4.jpeg)


总之，mysql主从模式默认是异步复制的，而MySQL Cluster是同步复制的，只要设置为相应的模式即是在使用相应的同步策略。
从MySQL5.5开始，MySQL以插件的形式支持半同步复制。其实说明半同步复制是更好的方式，兼顾了同步和性能的问题。