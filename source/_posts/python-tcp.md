---
title: python tcp
date: 2020-02-05 13:51:47
tags: python
---

> tcp 编程

<!-- more -->


### 1. client 客户端
```python
# 导入socket库:
import socket
# 用 AF_INET:ipv4 地址，SOCK_STREAM:tcp 协议
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
