---
title: TCP/UDP 协议
date: 2020-04-21 08:59:51
tags: Linux
---

> TCP 
> UDP

<!-- more -->


## 一. OSI 模型

1. 根据 OSI 模型，两个应用之间通信需要通过传输层，网络层，物理层；
2. 物理层是物理硬件，比如光纤，网线等，将 01 的状态通过载波传输到指定 MAC 地址。
3. 网络层的核心是 IP 协议，它将数据打包成 IP 数据报，通过数据分组和路由转发到指定 IP 地址。
4. IP 协议只是一个地址协议，并不保证数据包的完整。如果路由器丢包，需要发现丢了哪个包，以及重新发送这个包，此时就得依靠传输层的 TCP 协议。TCP 协议的主要作用是保证数据通信的完整性和可靠性，防止丢包。
5. 传输层主要协议包括 TCP 和 UDP，两者各有特性，合适不同的场景。

![](/img/2020/TCP_IP.png)


## 二. TCP 和 UDP 协议

#### 1. 主要区别

区别| TCP | UDP
-|-|-
定义|Transmission Control Protocol，传输控制协议|User Data Protocol，用户数据报协议
连接|必须先建立连接，即三次握手，才能发送数据|不需要建立连接，就能直接发送数据
模式|只能两个端点之间一对一地通信|支持一对一，一对多，多对一，多对多
可靠|提供可靠的交付服务，传输数据无差错，不丢失，不重复，且按时序到达|使用尽最大努力交付，但是不保证数据完整，有序。
数据|面向字节流，根据需要将数据拆分和合并后再发送|面向报文，一次就发送一个完整的报文
流量|有流量控制，根据网络情况调整数据传输速率|尽最大努力将应用程序的数据传输到网络上
通道|双向同时通信，即两端之间能互发信息|单向，比如客户端单向发送信息给服务端
头部|首部有20个字节 |首部开销小，只有8字节
主体|IP 报文长度 - 20 byte(IP 首部) - 20 byte(TCP 首部)|IP 报文长度 - 20 byte(IP 首部) - 8 byte(UDP 首部)



![](/img/2020/TCP_one.png)

![](/img/2020/UDP_one.gif)



#### 2. 使用场景
- TCP协议是基于连接的可靠协议，有流量控制和差错控制，也正因为有复杂的控制手段，所以传输效率比UDP低，合适对数据有准确性要求，效率不用太高的场景。
- UDP协议是基于无连接的不可靠协议，没有过多的控制和头部信息，只负责尽最快速度将数据发送，所以效率更高，合适容忍部分数据丢包，要求时效性较高的场景。


#### 4. TCP 用途
- 文件传输，FTP HTTP 对数据准确性要求高，速度可以相对慢
- 发送或接收邮件，POP IMAP SMTP 对数据准确性要求高，非紧急应用
- 远程登录，TELNET SSH 对数据准确性有一定要求，有连接的概念


#### 5. UDP 用途
- 在线视频，视频偶尔花了一个图像帧，人们还是能接受的
- 网络电话，语音数据包一般比较小，需要高速发送，偶尔断音或串音也没有问题
- 音乐播放，音频数据准确性和丢包要求比较低，但速度必须快，主持人口音不同步


## 三. TCP 报文解析

![](/img/2020/TCP_two.png)


- 源端口号, Source Port
长度为16位，指明主机发送数据的进程端口。

- 目的端口号, Destination Port
长度为16位，指明目的主机接收数据的进程端口。 

- 序列号, Sequence Number
长度为32位，用来标识从发送端向接收端发送的字节数量，可以理解成对字节的计数，确保发送数据的完整且有序。

- 确认号, Acknowledgement Number
长度为32位，用来标识接收端所期望收到的下一个序列号，即发送端上次发送的序列号 + 字节数，以便发送端知道接收端收到了哪些数据，确保发送数据不会丢失。确认号只有在 ACK 标志位为1时才有效。

- 首部长度
长度为4位，用于表示TCP报文首部的长度，首部前20个字节是必有的，后40个字节可能有。如果TCP报文首部是20个字节，则该位应是20/4=5。

- 保留位, Reserved
长度为6位，必须是0，它是为将来定义新用途保留的。

- 标志位, Flags
长度为6位，在TCP报文中不管是握手还是挥手还是传数据等，这6位标志都很重要，从左到右依次为：
	1. URG：紧急标志位，说明紧急指针有效；
	2. ACK：确认标志位，多数情况下空，说明序列号有效；
	3. PSH：推送标志位，置位时表示接收方应立即请求将报文交给应用层；
	4. RST：复位标志位，用于重建一个已经混乱的TCP连接；
	5. SYN：同步标志位，仅在三次握手建立TCP连接时有效，表示要建立连接；
	6. FIN：结束标志位，用于结束一个TCP会话。

- 窗口大小, Window Size
长度为16位，TCP流量控制由连接的每一端通过声明的窗口大小来提供。

- 检验和, Checksum
长度为16位，该校验值计算方式覆盖整个TCP报文数据，是个强制性的字段，是由发送端计算和存储，到接收端后，由接收端进行验证。

- 紧急指针, Urgent Pointer
长度为16位，指向数据中优先部分的最后一个字节，通知接收方紧急数据的长度，该字段在URG标志置位时有效。

- 选项, Options
长度为0-40个字节，必须以4个字节为单位变化，必要时可以填充 0，是TCP首部的扩展字段，不一定会有。

- 数据, data
具体传输的数据



## 四. UDP 报文解析

- 源端口号, Source Port
长度为16位，指明主机发送数据的进程端口。

- 目的端口号, Destination Port
长度为16位，指明目的主机接收数据的进程端口。

- UDP长度
长度为16位，该字段值为UDP首部和数据两部分的总字节数。

- 检验和, Checksum
长度为16位，UDP 数据检验值，由发送端计算和存储，由接收端校验。

- 数据, data
具体传输的数据


## 五. TCP 三次握手

![](/img/2020/TCP_three_handle.png)

#### 1. 握手目的
TCP 是面向连接的，一对一的，双向同时通信的可靠协议，必须在通信前，让 Client 和 Server 确认自己和对方的发送和接收通道正常。


#### 2. 握手过程

- 第一次握手：
	(1). Client 向 Server 发起连接请求，标志位 SYN = 1，序列号 Seq = x (一般是 0)；
	(2). Client 发送完毕后进入 SYN_SENT 状态，等待 Server 确认。
- 第二次握手：
	(1). Server 收到 Client 发起的标志位 SYN = 1 的连接请求后，返回一个确认响应，Flags 标志 SYN=1，ACK=1，序列号 Seq = y (一般是 0)，确认号 Ack = x + 1 ( Client 发送的序列号 Seq + 1)；
	(2). Server 发送完毕后进入 SYN_RECV(SYN_RCVD) 状态，等待 Client 回应。
- 第三次握手：
	(1). Client 收到 Server 返回的标志位 SYN = 1 响应后，检查返回的标志位 ACK = 1，以及确认号 Ack 是否等于第一次握手发送的序列号 Seq + 1，即 x + 1
	(2). 如果正确，那么 Client 返回一个响应给 Server ，标志位 ACK=1，确认号 Ack = y + 1 ( Server 发送的序列号 Seq + 1)
	(3). Client 发送完毕后，进入 ESTABLISHED 状态，准备开始发送数据给 Server。
	(4). Server 收到 Client 返回的标志位 ACK=1 响应后，检查确认号 Ack 是否等于第二次握手发送的序列号 Seq + 1，即 y + 1
	(5). 如果正确，那么 Server 进入 ESTABLISHED 状态，准备接受 Client 的数据。


#### 3. 为啥三次握手： 让 Client 和 Server 确认自己和对方的发送和接收通道都正常。

- 第一次握手，Server 收到了 Client 的连接请求，此时 Server 知道 Client 发送的通道正常，也知道自己的接收通道正常
- 第二次握手，Client 收到了 Server 的响应和连接请求，此时 Client 知道自己的发送和接收通道都正常，也知道 Server 的发送和接收通道都正常。
- 第三次握手，Server 收到了 Client 的响应，此时 Server 知道自己的发送通道正常，也知道 Client 接收的通道也正常


分类| 第一次握手 | 第二次握手| 第三次握手
-|-|-|-
Server 确认 Client 发送正常|Y|-|-
Server 确认 Server 接收正常|Y|-|-
Client 确认 Client 发送正常|-|Y|-
Client 确认 Client 接收正常|-|Y|-
Client 确认 Server 发送正常|-|Y|-
Client 确认 Server 接收正常|-|Y|-
Server 确认 Client 接收正常|-|-|Y
Server 确认 Server 发送正常|-|-|Y


## 六. TCP 四次挥手


![](/img/2020/TCP_bye_one.jpg)

#### 1. 挥手目的：
TCP 是双向同时通信的协议，当 Server 和 Client 之间的数据发送完毕后，需要结束两端的连接通道。

#### 2. 挥手过程：

- 第一次挥手：
	(1). Client 发送结束请求给 Server，标志位 FIN = 1，序列号 Seq = x 请求，表示 Client 关闭了 Client 到 Server 的数据传送通道；
	(2). Client 发送完毕后，进入 FIN_WAIT_1 状态。
- 第二次挥手：
	(1). Server 收到 Client 标志位 FIN = 1 的结束请求后，发送响应给 Client，标志位 ACK = 1，确认号 Ack = x + 1（ Client 发送的序列号 Seq + 1 ）；
	(2). Server 发送完毕后，进入 CLOSE_WAIT 状态，此时 Server 依然可以给 Client 发送数据。
	(3). Client 收到 Server 的结束确认后，检查标志 ACK = 1 且 Ack = x + 1，如果正确，则进入 FIN_WAIT_2 状态。
- 第三次挥手：
	(1). Server 发送所有数据给 Client 后，最终会发送结束请求给 Client，标志位 FIN = 1，序列号 Seq = y，表示 Server 关闭了 Server 到 Client 的数据传送通道；
	(2). Server 发送完毕后，进入 LAST_ACK 状态。
- 第四次挥手：
	(1). Client 收到 Server 标志 FIN = 1 的结束请求后，接着发回响应给 Server，标志位 ACK = 1 ，确认号 Ack = y + 1 （ Server 发送的序列号 Seq + 1 ）；
	(2). Client 发送完毕后，进入 TIME_WAIT 状态，此过程将会持续 2MSL，即两倍最长报文时间 Maximum Segment Lifetime，之后 Client 才进入 CLOSED 状态。
	(3). Server 收到 Client 的结束确认后，检查标志位 ACK = 1 且 Ack = y + 1，如果正确，则进入 CLOSED 状态。



#### 3. 为啥四次挥手：Client 发送 FIN 时，Server 可能还有数据，所以 Server 回应 Client 的 ACK 和 Server 的 FIN 分开两次发送。

- 三次握手过程中，第二次握手时，Server 将 Client 连接请求的确认标志位 ACK 和允许建立连接的标志位 SYN 与一起发送给 Client。
- 四次挥手过程中，Server 在收到 Client 的结束请求 FIN 后，可能还有数据要发送，所以会首先响应一个 ACK 给 Client，等数据全部发送完毕后，再发送一个 FIN 给 Client
- Client 发送自己连接的 FIN， Client 发送 ACK 响应 Server 的 FIN，发生了两次挥手。
- Server 发送自己连接的 FIN， Server 发送 ACK 响应 Client 的 FIN，也发送了两次挥手，所以一共就四次挥手
- 如果 Server 响应 Client 的 ACK 和自己结束的 FIN 一起发送给 Client，那么其实三次挥手就能完成了。



分类| 第一次挥手 | 第二次挥手| 第三次挥手|第四次挥手
-|-|-|-|-
Client 发起 Client 通道结束请求|Y|-|-|-
Server 响应 Client 通道结束请求|-|Y|-|-
Server 发起 Server 通道结束请求|-|-|Y|-
Client 响应 Server 通道结束请求|-|-|-|Y

![](/img/2020/TCP_bye_two.png)



#### 4. 为啥 Client 有 TIME_WAIT 状态，而且要等于 2MSL

- 确保最后一个 Client 确认结束报文 ACK 能够到达 Server，两端通道都可靠地结束。
	(1). 如果 Server 没收到 Client 发送来的确认报文，那么就会重新发送一次结束请求；
	(2). 如果此时 Client 处于 CLOSED 状态，那么 Server 将会收到 Client 标志位 RST = 1 的报文，表示需要重建已经错乱了的连接。
	(3). 而 Client 在状态 TIME_WAIT，依然能够接受确认报文，等待一段时间就是为了处理 Server 重发结束请求的情况。
- 等待一段时间是为了让本连接持续时间内所产生的所有报文都从网络中消失，避免端口相同的前后两个连接的报文混乱。
	(1). TCP 连接是根据端口区分不同连接的
	(2). 如果旧连接的数据在网络延迟才收到，而新连接使用了相同的端口，那么 Server 会以为相同的连接，那么旧连接的报文和新连接的报文就会混在一起了。
	(3). Client 等待两倍最长报文时间 Maximum Segment Lifetime，就是为了确保旧连接的报文，在网络中消失。


RFC793中规定MSL为2分钟，实际应用中常用的是30秒，1分钟和2分钟等



## 七. 大量 SYN_RECV 的 TCP 连接

#### 1. 原因
- 在第二次握手时，Server 在发送 SYN-ACK 给 Client 之后，会进入 SYN_RECV 状态；当收到 Client 响应的 ACK 后，即第三次握手时，Server 才转入 ESTABLISHED 状态。
- 如果 Client 在短时间内伪造大量不存在的IP地址，并向 Server 不断地发送 SYN 连接请求，由于IP源地址是不存在的，因此 Server 需要不断重发 ACK 直至超时，这些伪造的 SYN 请求将会占用未连接的队列，导致正常的 SYN 请求因为队列满而被丢弃，从而引起网络堵塞甚至系统瘫痪。

```shell
# netstat -nap | grep SYN_RECV
# netstat -n | grep tcp | awk '{print $6}' | sort | uniq -c | sort -nr
   1606 TIME_WAIT
    317 ESTABLISHED
      3 CLOSE_WAIT
# netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
TIME_WAIT 2448
CLOSE_WAIT 43
SYN_SENT 6
ESTABLISHED 1543
SYN_RECV 2
LAST_ACK 1

```

#### 2. 问题
- DOS, Denial of Service, 拒绝服务攻击，简单说就是发送大量请求是使服务器瘫痪，典型是 TCP 协议的 SYN FLOOD 洪水攻击
- DDOS, Distributed Denial of Service, 分布式拒绝服务攻击，在DOS攻击基础上发起的攻击，可以通俗理解，DOS 是单挑，而DDOS是群殴。


#### 3. 识别 DDOS
- 反欺骗：对数据包地址以及端口的正确性进行验证，同时进行反向探测
- 协议分析：每个数据包类型需要符合规范，只要不符合规范，就自动识别并将其过滤掉。
- 特征过滤：真实用户请求一般是随机访问的，如果是定时发起请求请求，有可能是模拟请求，可屏蔽掉这些用户IP

#### 4. 防范 DDOS

- 在机房入口拦截
 - 在机房网络入口架设防火墙，如果某个内网IP被 DDOS 攻击，直接丢掉这个IP的数据包，压根不会到到达内网服务器，这是机房运营商常用的防范措施。
 - 流量阀值控制，如果访问过大时，可以限制流量，或者检测工具，发现并过滤掉异常的流量，达到净化流量的功能，云服务商一般都支持流量清洗。
 

- 在本机防火墙拦截
 - 本机防火墙 iptables 根据异常请求的特征，直接过滤掉，比如 DROP 指定 IP
 
- 在 Web 服务器拦截
 - 依然是根据异常请求的特征过滤掉，Nginx 支持 deny 指定IP
 - 设置最大连接数据，Nginx 支持 limit connect 设置。

- 其他方式
 - 通过 CDN 分散请求，一定程度上能缓解服务器的压力，尤其是静态资源，但是对于动态请求，比如登录校验，用户评论等，主服务器也是扛不住攻击的。
 - 扩容，增加带宽，升级服务器，以便快速处理大量的请求，此方法费时费力，而且一般扛不住，DDOS 攻击本质就是通过大量请求，让你无法正常提供服务。



## 八. 大量 CLOSE_WAIT 的 TCP 连接

#### 1. 原因：
- 在 TCP 四次挥手中，当 Client 主动发送 FIN 请求，Server 会响应 ACK，然后进入 CLOSE_WAIT 状态，这只是表示 Client 到 Server 的通道关闭了，此时 Server 到 Client 的通道可能还有数据要发送。
- 当 Server 向 Client 发送完所有数据，就会发送自己的 FIN 请求给 Client，之后 Server 进入 LAST_ACK 状态。
- 如果 Server 出现大量的 CLOSE_WAIT，就表示 Server 有大量的 TCP 没有正常的发送自动的 FIN，即没有正常地自己关闭。
- 可能是 Server 的程序出现问题，socket accept后没有 close，文件 open 后没有 close

```shell
[root@ ~]# netstat | grep tcp |  awk '{print $6}' | sort |uniq -c | sort -nr
   5461 CLOSE_WAIT
    367 ESTABLISHED
      2 LAST_ACK
      1 FIN_WAIT2
```

#### 2. 问题
由于 Server 没有正常关闭 TCP 连接，则该连接使用的句柄将会无法释放，而每个进程能打开的句柄数是有限的，所以会引发 too many open files in system 的问题。
- 查看进程打开的句柄数

```shell
# 数字为进程ID
# 对于当前进程，0 是标准输入，1 是标准出，2 是错误输出
# 其他数字是进程打开的句柄，比如引用的lib文件，打开的 socket 通道等
[root@ ~] ls -l /proc/13574/fd
lr-x------ 1 root root 64  5月 15 15:09 0 -> /dev/null
l-wx------ 1 root root 64  5月 15 15:09 1 -> /data/home/test_user/log/output.log
l-wx------ 1 root root 64  5月 15 15:09 2 -> /data/home/test_user/log/error.log
l-wx------ 1 root root 64  5月 15 15:09 3 -> /data/home/test_user/lib/jul-to-slf4j-1.7.25.jar
lrwx------ 1 root root 64  5月 15 15:09 4 -> socket:[1236791109]
```

- 查看进程可打开的最大句柄数

```shell
[root@ ~] ulimit -n
1024
# 或者
[root@ ~] ulimit -a
····
open files                      (-n) 1024
····
```


#### 3. 处理建议
(1) 增加进程的最大文件句柄数

```shell
# 临时生效
# H=hard 硬限制，实际限制
# S=soft 软限制，只是给出告警，但不限制
[root@ ~] ulimit -SHn 65535

# 永久生效
# 修改 /etc/security/limits.conf 
# * 表示所用的用户
[root@ ~] echo "* soft nofile 65535"  >> /etc/security/limits.conf
[root@ ~] echo "* hard nofile 65535"  >> /etc/security/limits.conf
```

(2) 如果是超过了系统级别的句柄限制，则修改系统总限制

```shell
# 临时生效
# 重启后会失效
[root@ ~] echo  6553560 > /proc/sys/fs/file-max

#永久生效
# 修改 /etc/sysctl.conf
# 重启生效
echo "fs.file-max=6553560" >> /etc/sysctl.conf
```

(3) 检查 Server 程序，是否存在没有关闭句柄的BUG，以python代码为例

```python
# 接收请求，打开 socket 句柄
sock, addr = s.accept()
# 最后是否关闭句柄了
sock.close()

# 通过 with 打开文件和自动关闭
with open('hello.txt','r') as f:
    print(f.readlines())
```

## 九. 大量 LAST_ACK 的 TCP 连接
#### 1. 原因
- 在 TCP 四次挥手中，Client 主动发送 FIN 给 Server，Server 最后也会发送 FIN 给 Client，接着 Server 进入 LAST_ACK 状态，等待 Client 回复 ACK。
- 如果网络原因 Client 没有收到 Server 的 FIN，则会触发 TCP 重传机制，Server 会再次发送 FIN，直到重传时，然后 Server 就会进入 CLOSED 状态。
- 如果超过 2MSL 后，Client 进入 CLOSED 了才收到 Server 的 FIN，则会回复 RST 包给 Server，表示本次 TCP 连接已经出现错乱了，此时 Server 也会进入 CLOSED 状态。

#### 2. 问题
- LAST_ACK 状态下，TCP 连接依然占用句柄，无法释放，也会触发 too many open files in system。

#### 3. 处理建议
TCP 的重传 FIN 机制能有效地确保链接两端都可靠地结束，而且重传机制超时会进入 CLOSED 状态，所以很多情况下， LAST_ACK 状态只是短暂存在，很快消失，无需太多干预。


## 十. 大量 TIME_WAIT 状态的 TCP 连接
#### 1. 原因
- 在 TCP 四次挥手中，TIME_WAIT 是主动关闭方的状态，比如上面提到的 Client 主动向 Serve 发送 FIN，最后 Client 会进入 TIME_WAIT 状态，等待 2 MSL 后，进入 CLOSED。
- 利用 Nginx 转发请求到 PHP-FPM ，如果 PHP-FPM 执行时间太长，Nginx 判断超时了，就会主动发送 FIN ，此时 Nginx 充当 Client 角色，就会出现 TIME_WAIT 的连接。
- HTTP 1.0 采用短连接，所以 Nginx 返回内容给浏览器时，会自动断开连接，此时 Nginx 充当 Server 角色，主动发送 FIN，也会出现  TIME_WAIT 的连接。
- PHP-FPM 机器会发起很多 DB 连接请求到 MySQL 服务器，如果此时 MySQL 服务器出现异常，导致连接请求超时， PHP-FPM 会主动断开，此时 PHP-FPM  充当  Client 角色，就会出现 TIME_WAIT 的连接。
- MySQL 服务器在收到客户端 quit 的命令后，会主动关闭客户端，此时 MySQL 服务器 充当 Server 角色，也有可能出现 TIME_WAIT 状态，采用类似机制的还有 Redis 服务器。

#### 2. 问题
- TIME_WAIT 状态下，TCP 连接依然占用句柄，无法释放，也会触发 too many open files in system。
- 对于主动发起TCP连接，并主动关闭的一方，TIME_WAIT 状态会一直占用系统的源端口，但是系统端口是有限的。
- 所有连接会保存到系统的 hask table，一直不释放的连接会一直占用内存，但是相对较少，影响不大。
- 新连接每次遍历空闲随机端口耗费的时间也会变长，但是对CPU影响也是很小的。

#### 3. 解决方法
TIME_WAIT 状态下的 TCP 连接将会在 2 MSL 后自动转入 CLOSED，即最终会消失的，如果系统句柄没有超限，源端口足够，也无需太多干预。
如果确实影响了正常服务，可按照以下顺序处理，其中(4),(5),(6)方法都违背了 TCP 协议规范，非必要情况，都不推荐：

(1) 将短连接改成长连接，减少连接数量，在高并发下，长连接的性能会优于短连接。
(2) Server 和 Client 端，增加句柄数，请参考上面提到的解决 CLOSE_WAIT 的方法
(3) Client 端，增加源端口数，系统的源端口在启动时设置了范围
查看本地源端口范围

```shell
[root@ ~]# sysctl -a | grep ip_local_port_range
net.ipv4.ip_local_port_range = 32768	60999
````

修改本地源端口范围

```shell
# 临时生效
# 重启后会失效
[root@ ~] echo  "30000 63000" > /proc/sys/net/ipv4/ip_local_port_range

#永久生效
# 修改 /etc/sysctl.conf
# 重启生效
echo "net.ipv4.ip_local_port_range=30000 63000" >> /etc/sysctl.conf
```

(4) Server 和 Client 端，修改 TIME_WAIT 状态连接数量上限 tcp_max_tw_buckets，超过上限的 TIME_WAIT 连接自动转入 CLOSED，会有一定安全隐患，参考等待 2 MSL 的作用。

(5) Client 端，开启 tcp_tw_reuse (默认关闭)，即复用 TIME_WAIT 状态的连接，此时会存在延迟数据和新连接数据冲突的问题，所以必须借助 Server 和 Client 端的参数 tcp_timestamps (默认开启) 来标记不同时间的连接。该参数只对 Client 端有用，Server 端不需要开启，即使 Server 端需要连接 DB，一般也是长连接，所以不会有大量的连接。

(6) Server 和 Client 端，开启 tcp_tw_recycle (默认关闭)，即回收 TIME_WAIT 状态的连接，比 2 MSL 更短的时间回收，此参数也要依赖 tcp_timestamps 参数；如果 Client 处于 NAT 网络中，TCP 头部的时间戳即 tcp_timestamps 的标记可能会异常 ，导致TCP连接建立错误，所以开启要慎重。


## 十一. TCP粘包、拆包

#### 1. UDP 是基于报文发送的，一个 UDP 报文就是完整的数据，UDP 首部采用了 16bit 来指示 UDP 数据报文的长度，因此在应用层能很好的将不同的数据报文区分开，从而避免粘包和拆包的问题。

#### 2. TCP 粘包、拆包的原因
- TCP 是基于字节流的，虽然应用层和 TCP 传输层之间的数据交互是大小不等的数据块，但是 TCP 并没有把这些数据块区分边界，仅仅是一连串没有结构的字节流；
- TCP 的首部没有表示数据长度的字段，基于上面两点，在使用 TCP 传输数据时，才有粘包或者拆包现象发生的可能。


#### 3. TCP 粘包、拆包的场景
- 要发送的数据小于 TCP 发送缓冲区的大小，TCP 将多次写入缓冲区的数据一次发送出去，将会发生粘包。
- 要发送的数据大于 TCP 发送缓冲区剩余空间大小，将会发生拆包。
- 待发送数据大于 MSS（最大报文长度），TCP 在传输前将进行拆包。
- 接收数据端的应用层没有及时读取接收缓冲区中的数据，将发生粘包

![](/img/2020/TCP_pkg_one.jpg)
![](/img/2020/TCP_pkg_three.jpg)
![](/img/2020/TCP_pkg_two.jpg)


#### 4. TCP粘包、拆包的解决办法
由于 TCP 本身是面向字节流的，无法理解上层的业务数据，所以在底层是无法保证数据包不被拆分和重组的，这个问题只能通过上层的应用协议栈设计来解决。

- 消息定长：发送端将每个数据包封装为固定长度（不够的可以通过补 0 填充），这样接收端每次接收缓冲区中读取固定长度的数据就自然而然的把每个数据包拆分开来。
- 设置消息边界：服务端从网络流中按消息边界分离出消息内容。在包尾增加回车换行符进行分割，例如 FTP 协议。
- 将消息分为消息头和消息体：消息头中包含表示消息总长度（或者消息体长度）的字段。
- 更复杂的应用层协议，比如 Netty 中实现的一些协议都对粘包、拆包做了很好的处理。

