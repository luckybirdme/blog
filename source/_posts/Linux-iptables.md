---
title: Linux iptables
date: 2020-05-14 08:22:15
tags: Linux
---

> iptables

<!-- more -->

### 一. iptables 由来

1. iptables 其实不是真正的防火墙，它只是一个命令工具，位于用户空间，用户通过 iptables 命令，将用户的安全设定执行到对应的"安全框架"中，这个"安全框架"才是真正的防火墙，这个框架的名字叫 netfilter。
2. netfilter 才是防火墙真正的安全框架，它位于内核空间，提供了网络地址转换(Network Address Translate)，数据包内容修改，以及数据包过滤的功能。
3. iptables 主要工作在OSI 的第二(链路层)，三(网络层)，四层(传输层)

iptables 流程图

![](/img/2020/Linux_iptables.png)


### 二. iptables 链 

##### 1. 定义：
数据包从网卡到本地应用，应用处理完后数据包，再从本地应用到网卡，需要经过防火墙的多个链路 (chain)，包括：
- PREROUTING, 从网卡进入到内核空间
- FORWARD, 从一个网卡转发到另一个网卡
- INPUT, 从内核空间到用户空间，即应用程序
- OUTPUT, 从用户空间到内核空间
- POSTROUTING, 从内核空间到网卡

![](/img/2020/iptables_chain.png)


### 三. iptables 表

##### 1. 定义：
在每个防火墙的链路，可以对数据包设置多种类型的过滤和修改规则，每种类型的规则保存在规则表 (table)，包括：
- raw 表：用来决定是否对数据包进行状态跟踪，不常用
- mangle 表：为数据包设置标记，有ACK、SYN、FIN、RST、PSH、URG等标记，不常用
- nat 表：修改数据包的IP地址、端口等信息，常用于网关型防火墙
- filter 表：过滤指定IP和端口的数据包，经常使用，也是默认的规则表

![](/img/2020/iptables_table.png)


##### 2. 使用
由于防火墙不同的链路位于不同的网络层次，所以每个链路只有指定部分规则表生效，具体如下

分类|raw|mangle|nat|filter
-|-|-|-|-
PREROUTING|Y|Y|Y|-
FORWARD|-|Y|-|Y
INPUT|-|Y|-|Y
OUTPUT|Y|Y|Y|Y
POSTROUTING|-|Y|Y|-


### 四. iptables 命令
##### 1. 匹配原理
- 从上到下遍历规则，如果匹配到其中一条规则，则立即处理并返回，不再对后续规则进行匹配。
- 如果没有匹配到任何规则，那么将会使用默认的规则，可能是直接丢弃数据包，也可能直接放过。


##### 2. 命令格式
- 常用格式

```
iptables [-t 表名] 命令选项 [链名] [规则号] [条件匹配] [-j 目标动作]
```

- 表名，如果没有指定默认 filter 表

```shell
iptables -t nat
```

- 命令选项，包括规则的增删改查等

```shell
# 在末尾追加规则
iptables -t filter -A
# 插入指定规则行，默认是首部，也可以指定行
iptables -t filter -I
# 删除指定规则，需指明规则序号或指明规则本身
iptables -t filter -D
# 替换规则，需指明规则序号或指明规则本身
iptables -t filter -R
# 清空指定规则
iptables -t filter -F
# 显示规则
iptables -t filter -L 
```


- 链名，就是上面提到的防火墙链路 (chain)

```shell
iptables -t nat -A PREROUTING
iptables -t filter -A FORWARD
iptables -t filter -A INPUT
iptables -t filter -A OUTPUT
iptables -t nat -A POSTROUTING
```

- 规则号，就是当前需要操作的规则行，从上往下数 1,2,3 ···


```shell
# 插入到第三行，原来第三行下移到第四行
iptables -t filter -I INPUT 3
```

- 匹配条件，主要包括五元组信息，通信协议，源IP，源端口，目的IP，目的端口

```shell
# 匹配目的端口是 3306 ，协议是 TCP 的数据包
iptables -t filter -I INPUT -p tcp --dport 3306
```

- 动作，匹配到数据包的动作

```shell
# 匹配目的端口是 3306 ，协议是 TCP 的数据包，直接丢弃
iptables -t filter -I INPUT -p tcp --dport 3306 -j DROP
# 常用动作
ACCEPT：放开
DROP：丢弃
REJECT：拒绝
REDIRECT：端口重定向；
LOG：记录日志；
DNAT：目的地址转换
SNAT：源地址转换；
```

### 五. iptables 使用


##### 1. 屏蔽本机高危端口，防止信息泄露

```shell
iptables -t filter -I INPUT -p tcp --dport 3306 -j DROP
```

##### 2. 只放通指定IP，限制来源用户

```shell
# 插入第一行，指定IP段允许访问
iptables -t filter -I INPUT -p tcp -s 10.10.10.0/24 -j ACCEPT
# 末尾追加兜底规则，屏蔽所有数据包
iptables -t filter -A INPUT -p tcp -j DROP
```

##### 3. 路由转发，搭建 NAT (Network Address Translation) 网络
- 先经过 NAT table 的 PREROUTING 链；
- 经由路由判断确定这个封包是要进入本机与否，若不进入本机，则下一步；
- 再经过 Filter table 的 FORWARD 链；
- 通过 NAT table 的 POSTROUTING 链，最后传送出去。

```shell
# 开启路由转发，临时生效；永久生效请修改 /etc/sysctl.conf 文件
echo 1 > /proc/sys/net/ipv4/ip_forward

# 让内网机器，通过 NAT 机器，访问外网，所以将内网机器的IP，即源IP，改成 NAT 机器的IP
iptables –t nat –A POSTROUTING –s 192.168.1.5 –o eth1 –j SNAT --to-source 111.196.221.212

# 让外部机器，通过 NAT 机器，访问内网，所以将目的机器的IP，改成内网机器的IP
iptables –t nat –A PREROUTING –i eth1 –d 111.196.221.212 –p tcp –dport 80 –j DNAT --to-destination 192.168.1.5:80

```

### 六. iptables 备份

##### 1. 备份

```shell
iptables-save >/PATH/TO/SOME_RULE_FILE
```

##### 2. 恢复

```shell
iptables-restore < /PATH/FROM/SOME_RULE_FILE
```