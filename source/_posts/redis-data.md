---
title: Redis 数据备份和恢复
date: 2020-04-07 08:14:08
tags: Redis
---

> 本文基于 Redis 4.0 版本

<!-- more -->

## 一. RDB
对内存中数据库状态进行快照（snapshotting），生成 RDB 文件
#### 1. 备份命名
- 手动：通过手动执行 save 或者 bgsave 命令
- 自动：通过 redis.conf 配置触发调用 bgsave 命令
```sh
#只要满足以下三个条件中的其中一个，BGSAVE命令就会被执行
#服务器在 900 秒之内，对数据库进行了 1 次修改
save 900 1
#服务器在 300 秒之内，对数据库进行了 10 次修改
save 300 10
#服务器在 60 秒之内，对数据库进行了 10000 次修改
save 60  10000
```

#### 2. 备份过程
- save : Redis 主进程会 fork 子进程进行内存数据复制，在复制完成后才一次性替换 RDB 文件，所以 RDB 文件内容是 fork 那一刻的内容，RDB 文件任何时刻的数据都是完整备份的。
- bgsave : 跟 save 类似，唯一区别在于 save 过程会阻塞 Redis 主进程执行其他数据操作，而 bgsave 是在后台运行的，不阻塞主进程。


#### 3. 恢复数据
Redis启动后会读取RDB快照文件，将数据从硬盘载入到内存，大小为2GB的快照文件载入到内存中需花费 1 分钟左右，但也要看机器性能以及数据类型。

#### 4. 优点
- RDB 文件内容是经过压缩保存的，所以体积小，传输快，合适快速恢复大数据集。

#### 5. 缺点
RDB 采用快照方式备份数据，此时写入的数据将会丢失，无法保证数据的完整性。



## 二. AOF
以追加形式，把每条写命令都写入 AOF 文件，作为数据修改记录文件
#### 1. 备份命名
在配置文件 redis.conf 设置 AOF 的策略，因为性能损耗，默认是不开启的
```sh
# 开启 AOF
appendonly yes
# 每提交一个修改命令都调用fsync到AOF文件，非常慢，但是很安全
appendfsync always
# 每秒都调用fsyns刷新到AOF文件，很快但可能丢失一秒内的数据；
appendfsync everysec
# 依靠OS进行刷新(默认 30 秒)，redis不主动刷新AOF，这样最快但是安全性差；
appendfsync no
# 配置重写触发机制（当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发）
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

#### 2. 备份过程
开启AOF持久化后每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬盘中的AOF文件。
如果觉得以前的数据操作记录太臃肿，可以通过 bgrewriteaof 命令重写 AOF 文件，跟 bgsave 一样通过子进程完成，不阻塞主进程。

#### 3. 数据恢复
在启动时Redis，载入AOF文件，创建模拟客户端，从AOF文件中读取一条命令，使用模拟客户端执行命令，循环读取并执行命令，直到全部完成，载入的速度相较RDB会慢一些。而且如果 AOF 和 RDB 文件同时存在，将会优先执行 AOF 文件。

#### 4. 优点
- 通过 appendfsync 配置策略，可以确保数据完整性。

#### 5. 缺点
- 虽然有重写机制，但是日志方式保存所有操作命令，文件体积会比 RDB 大。
- 因为文件大，而且存在重复执行命令，所以数据恢复比 RDB 会慢好多。
- 如果每条命令触发一次持久化到 AOF 文件，对机器性能损耗有点大。

## 三. 混合备份模式
混合备份模式先生存 RDB 文件，再将缓存命令生成 AOF 文件，然后合并新的 AOF 文件。
#### 1. 备份命名
在配置文件 redis.conf 设置混合备份模式，默认是不开启的
```sh
# 开启
aof-use-rdb-preamble yes
```

#### 2. 备份过程
先通过 bgsave 命令将内存全量数据生成 RDB 文件，再将 RDB 文件生成过程中客户端发起的命令从缓冲区写到 AOF 文件，最后将 RDB 文件 和 AOF 文件合并成新的 AOF 文件。新的 AOF 文件前部分是 RDB 内容，后部分是 AOF 新增内容。


#### 3. 数据恢复
跟前面 AOF 文件一样的恢复方式，只是会判断 AOF 前部分内容是否为 RDB 格式，如果是则采用 RDB 格式读取和写入到内容，最后再把末尾 AOF 格式的数据写入内容。


#### 4. 优点
- 结合了RDB持久化 和 AOF 持久化的优点, 由于绝大部分都是RDB格式，加载速度快，同时结合AOF，增量的数据以AOF方式保存了，数据更少的丢失

#### 5. 缺点
- 兼容性差，一旦开启了混合持久化，在4.0之前版本都不识别该aof文件，同时由于前部分是RDB格式，阅读性较差