---
title: OSI 链路层
date: 2020-05-17 21:52:02
tags: Linux
---

> 链路层

<!-- more -->


### 1. 位置
链路层位于 OSI 的第二层

### 2. 协议
- 以太网 (Ethernet,IEEE 802.3)，目前局域网的主流协议。
- 点对点协议 (Point to Point Protocol)，缩写为PPP，即拨号上网。目前广域网上应用最广泛的协议之一，它的优点在于简单、具备用户验证能力、可以解决IP分配等。

### 3. 数据
用以太网为例子

![](/img/2020/data_link_one.jpg)

- 目的 MAC 地址，目的主机的 MAC 地址
- 源 MAC 地址，源主机的 MAC 地址
- 类型，上层(网络层)协议，比如 IP 协议， ARP 协议等
- 数据，要传输的数据，不过该数据最长为 1500 个字节，该长度称为最大传输单元(MTU) ，即 Maximum Transmission Unit。
- 末尾追加结束校验符。 


### 4. MTU (1500 Byte) 的影响
- MTU对IP协议的影响
（1）IP报文在超过MTU后需要分片，接收端需要组装；
（2）一旦分片后的IP报文有一部分丢失，则接收端组装会失败，对于整个P报文而言相当于传输失败，而IP协议不会负责重新传输数据；
（3）由于MTU影响的IP报文的分片和组装会加大报文丢失的可能性；
（4）报文的分片和组装由IP层自己做，会加大传输的成本，降低性能。

- MTU对UDP协议的影响
（1）UDP协议的报头为固定的20字节；
（2）若UDP数据的长度超过（1500-20）1480字节，则数据在网络层会分片；
（3）数据的分片会加大数据丢失的可能性。

- MTU对TCP协议的影响
（1）TCP协议的报头长度为20–60字节；
（2）若TCP报文的总长度超过1500字节，则数据同样在网络层会分片；
（3）TCP单个数据报的最大长度称为最大段尺寸MSS；
（4）在TCP三次握手建立连接的时候，双方会商量传输中MSS的大小；
（5）与UDP相同的是，分片越多数据丢包的可能性越大，可靠性也就越差。


### 5. 具体实现
集线器，二层交换机