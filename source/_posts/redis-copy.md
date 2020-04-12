---
title: Redis 主从同步
date: 2020-04-09 07:29:17
tags: Redis
---

> 本文基于 Redis 4.0 版本
> 为了满足大量请求的读写，Redis 提供了一主多从的机制，以便进行读写分离，减轻压力

<!-- more -->

## 一. 同步机制
#### 1. Redis 支持一主多从或者级联的机制，主为 master，从为 slave，而且一个slave只能有一个master，如下图所示。
![](http://qiniucdn.luckybird.me/blog/img/2020/Redis-rsync.png)

#### 2. Redis 同步机制主要包括三部分：全量同步，部分同步，命令传播


## 二. 全量同步
#### 1. 触发时机
- 当 salve 刚启动初始化时，master 会复制一份完整的数据给 salve
- 当 slave 长时间掉线，master 无法得知 salve 的数据状态，也会全量同步一次

#### 2. 同步过程
- 从库向主库发起同步请求 psync
- 主库收到请求后通过 bgsave 命令生成 RDB 文件，同时缓存此过程的命令
- 主库向从库发送 RDB 文件
- 从库收到 RDB 文件后，加载数据到内存
- 主库向从库发送缓存命令
- 从库执行主库发送的缓存命令

![](http://qiniucdn.luckybird.me/blog/img/2020/Redis-full-psync.png)


## 三. 部分同步
master 每次同步数据到 slave ，都会记录一个 replication-id ，以此标记 salve 的数据状态。
之后 master 产生新命令时，会保存一份到「复制积压缓冲区」backlog 里面，同时记录 offset，以便标记增量更新的内容。
#### 1. 触发时机
- master 存在 salve 的唯一 id，即之前认证同步过了
- salve 的 <replication-id> 是 master 的 replication-id，即依然数据记录状态一致
- slave 给的 <offset> 在 master 的「复制积压缓冲区」backlog 里面，即 master 新产生的数据变化依然保留在缓冲区

#### 2. 同步过程
- slave 通过 PSYNC <replication-id> <offset> 命令，向 master 发起「部分重同步」请求。
- 如果 salve 的 id, replication-id, offset 满足条件, master 返回 CONTINUE 告知 slave 同意执行「部分重同步」
- master 将 backlog 中缓冲的命令发送给 slave（根据 slave 给的 offset）。
- slave 收到后，逐个执行这些命令。


## 四. 命令传播
当全量同步或者部分同步完成后，进入命令传播阶段
#### 1. 触发时机
master 的数据库状态被修改时，将导致变更的命令传播给 slave，从而让 slave 的数据库状态与 master 保持一致。

#### 2. 同步过程
- master 执行 client 命令，更改数据
- master 将新执行的命令写入到「复制积压缓冲区」backlog 里面
- master 通过异步进程将更新的数据发送给所有从服务器

 
## 五. 安全问题
#### 1. master 如果没有开启持久化，在宕机重启时，会丢失所有数据，此时同步机制会到所有 slave 的数据被清空，所以 master 尽量不要设置自动重启



