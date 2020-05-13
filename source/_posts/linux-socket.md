---
title: Linux socket 通信
date: 2020-05-11 07:51:19
tags: Linux
---

> socket

<!-- more -->


## 一. 定义
套接字（socket）是一个抽象层，应用程序可以通过它发送或接收数据，可对其进行像对文件一样的打开、读写和关闭等操作。套接字允许应用程序将I/O插入到网络中，并与网络中的其他应用程序进行通信。

## 二. 分类
为了满足不同的通信程序对通信质量和性能的要求，一般的网络系统提供了三种不同类型的套接字，以供用户在设计网络应用程序时根据不同的要求来选择。

#### 1. 流式套接字(SOCK-STREAM)。它提供了一种可靠的、面向连接的双向数据传输服务，实现了数据无差错、无重复的发送。流式套接字内设流量控制，被传输的数据看作是无记录边界的字节流。在TCP/IP协议簇中，使用TCP协议来实现字节流的传输，当用户想要发送大批量的数据或者对数据传输有较高的要求时，可以使用流式套接字。

#### 2. 数据报套接字(SOCK-DGRAM)。它提供了一种无连接、不可靠的双向数据传输服务。数据包以独立的形式被发送，并且保留了记录边界，不提供可靠性保证。数据在传输过程中可能会丢失或重复，并且不能保证在接收端按发送顺序接收数据。在TCP/IP协议簇中，使用UDP协议来实现数据报套接字。在出现差错的可能性较小或允许部分传输出错的应用场合，可以使用数据报套接字进行数据传输，这样通信的效率较高。

#### 3. 原始套接字(SOCK-RAW)。该套接字允许对较低层协议（如IP，ICMP协议）进行直接访问，常用于网络协议分析，检验新的网络协议实现，也可用于测试新配置或安装的网络设备。


![](/img/2020/Linux_socket.png)


## 三. 使用

以 Python 的 socket 类库为例子，感受 socket 的使用

### 1. client 客户端
```python
# 导入socket库:
import socket
# 用 AF_INET:ipv4 地址，SOCK_STREAM: 流式套接字(SOCK-STREAM), tcp 协议
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 建立连接:
s.connect(('127.0.0.1', 9999))
# 接收欢迎消息:
print(s.recv(1024).decode('utf-8'))
for data in [b'Michael', b'Tracy', b'Sarah']:
    # 发送数据:
    s.send(data)
    print(s.recv(1024).decode('utf-8'))

# 自定义数据结束标志，以便服务端关闭连接
s.send(b'exit')
# 关闭连接
s.close()

```


### 2. server 服务端

```python

# 导入socket库:
import socket
import time, threading

# 创建一个socket:
# 用 AF_INET:ipv4 地址，SOCK_STREAM:tcp 协议
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 监听端口:
s.bind(('127.0.0.1', 9999))
# 设置连接数
s.listen(5)
print('Waiting for connection...')

def tcplink(sock, addr):
    print('Accept new connection from %s:%s...' % addr)
    # 发送数据
    sock.send(b'Welcome!')
    while True:
        # 接收数据，限定大小，避免溢出
        data = sock.recv(1024)
        time.sleep(1)
        print('receive, %s!' % data.decode('utf-8'))
        # 判断客户端是否关闭
        if not data or data.decode('utf-8') == 'exit':
            break
        sock.send(('Hello, %s!' % data.decode('utf-8')).encode('utf-8'))
    # 关闭连接
    sock.close()
    print('Connection from %s:%s closed.' % addr)

while True:
    # 接受一个新连接:
    sock, addr = s.accept()
    # 创建新线程来处理TCP连接，以便接收多个连接
    t = threading.Thread(target=tcplink, args=(sock, addr))
    t.start()

```