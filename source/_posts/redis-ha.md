---
title: Redis 高可用
date: 2020-04-11 10:33:18
tags: Redis
---

> 本文基于 Redis 4.0 版本

<!-- more -->

## 一. 面临情况
##### 1. 不清楚 Redis 运行状态
##### 2. 单机无法确保服务高可用

## 二. Redis 高可用机制：Sentinel(哨兵)模式
![](http://qiniucdn.luckybird.me/blog/img/2020/Redis_Sentinel.png)

- 监控（Monitoring）：Sentinel会不断的检查你的主节点和从节点是否正常工作。
- 通知（Notification）：被监控的Redis实例如果出现问题，Sentinel可以通过API（pub）通知系统管理员或者其他程序。
- 自动故障转移（Automatic failover）：如果一个master离线，Sentinel会开始进行故障转移，master下的一个slave会被选为新的master，其他的slave会开始复制新的master。应用可以通过Redis服务的通知机制更新新的master地址。
- 配置提供者（Configuration provider）：客户端可以把Sentinel作为权威的配置发布者来获得最新的master地址。如果发生了故障转移，Sentinel集群会通知客户端新的master地址，并刷新redis的配置。



## 三. 优缺点
##### 1. 优点
- 监控 Redis 集群状态，一有故障可及时通知
- 节点出现故障通知 client 更换 master 地址，确保服务高可用

##### 2. 优点
- 集群部署相对复杂
- 从节点空闲浪费