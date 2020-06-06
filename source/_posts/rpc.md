---
title: RPC
date: 2020-06-05 23:18:24
tags: Linux
---

> RPC

<!-- more -->

## 一. 背景
##### 1. 进程内调用
因为进程内共享上下文，比如进程内存地址，所以直接调用对应方法的指针地址即可。

```python
def local_func(input):
    output='receive : ' + input 
    return True,output

state, response = local_func('hello')
if state:
    print(response)
```

##### 2. 进程间调用(IPC，Inter process communication)
进程是内存分配资源的最小单位，所以进程之间使用不同的内存地址，因此进程间调用，无法直接引用对方的内地地址。
进程间调用，也称为进程间通信，常用方式包括：

- 本地管道
- 本地共享内存
- 本地进程信号量
- 本地套接字 socket
- FIFO(First Input First Output)存储器
- 第三方消息队列
- 远程 RESTful API
- 远程过程调用 RPC(Remote Procedure Call)


其中，根据进程所在位置，可分为两大类

- 本地进程调用 LPC(Local Procedure Call)，比如本地管道，共享内存，本地套接字，FIFO 存储器等，需要通信的进程都在同一台物理机器上。
- 远程过程调用 RPC(Remote Procedure Call)，进程部署在不同物理机器上提供分布式服务，远程 RESTful API 也可归为此类。

## 二. RESTful API

##### 1. 定义
RESTful API 是一种网络应用程序的设计风格和开发方式，运行于OSI第七层(应用层)，基于HTTP/HTTPS，主要使用 XML 或 JSON 数据格式进行交互。

##### 2. 流程
![title](/img/2020/RPC_1.png)

##### 3. 方法
提供 HTTP 协议的主要方法，并对方法功能进行了明确定义。

- GET, 获取资源
- POST, 增加资源
- PUT, 修改资源
- DELETE, 删除资源
- HEAD, 获取资源元数据信息，比如是否存在，内容长度
- OPTIONS, 获取资源所支持类型，比如是否支持 PUT, DELETE 方法

## 三. 远程过程调用

##### 1. 定义
RPC 是一种通过网络从远机器上请求程序服务的协议，运行于OSI第四层(传输层)，基于TCP或UDP协议，主要采用 Client/Server 模式，常见数据交互格式 protocol buffer

##### 2. 流程

![title](/img/2020/RPC_2.png)

##### 3. 方法
RESTful API 基于 HTTP 协议定义方法，RPC 则是基于 ID 和方法之间映射来定义，调用时传递方法 ID，就会调用对应的方法，因此 RPC 的方法和作用可自由定义。

##### 4. 数据
protocol buffer 跟 XML、JSON一样，都是结构数据序列化的工具，实现数据交换和存储功能，主要区别如下：

协议| XML|JSON|Protocol Buffer
-|-|-|-
数据内容| 易读的 TAG 标签 | 易读的数组形式 | 二进制流，不方便阅读
数据大小|最大|较大|比前者小3到10倍
整体性能|一般|一般|比前者快20到100倍
格式转换|原始内容即可转换|原始内容即可转换|需要借助数据格式文件 proto


##### 5. 网络
RPC 大部分采用 TCP/UDP 协议，工作在第四层(网络层)，比 HTTP/HTTP2 协议性能更高。
也存在部分 RPC 采用 HTTP/HTTP2 协议，比如 gRPC, Netty。


##### 6. C/S 模式


```python
# client
state, response = remote_server_func(server_addr,func_id,param)

# server
func_id, param = accpet_from_client()
```

Client 端 

- 调用 remote_server_func , 并传递方法 ID, 即 func_id，以及参数 param
- 将 func_id 和 param 数据进行序列化，以二进制形式打包。
- 通过网络 socket，将数据包发送给 server_addr
- 等待服务器 server_addr 返回结果
- 如果服务器 server_addr 调用成功并返回结果，则将结果反序列化，赋值给 response

Server 端

- 在本地维护 func_id 和 函数的映射表 func_id_map，一般用哈希表
- 等待 Client 的请求到来
- 得到一个 Client 请求后，将其数据包反序列化，得到 func_id 和 param
- 在 func_id_map 中查找，得到对应的函数
- 传递 param 参数给对应函数，并执行该函数。
- 函数执行完后，将结果序列化，再通过网络 socket 返回给 Client


##### 7. 部署架构
为了方便 Client 调用 Server，一般 Server 会为 Client 提供本地调用的接口 API，这些接口会自动将数据序列化和反序列化，然后与 Server 通信，Client 只需要简单调用接口，不太需要关心底层，这种部署方式也称为动态代理模式。

![title](/img/2020/RPC_3.png)


Client 端代码架构

![title](/img/2020/RPC_4.png)

Server 端代码架构

![title](/img/2020/RPC_5.png)


## 四. 使用场景分析

##### 1. 远程调用主要解决问题

- 将服务部署在不同物理机，提供分布式服务。 
- 细分不同服务，便于管理和性能提升。

##### 2. RESTful API 和 RPC 对比

- RESTful API 采用 HTTP/HTTP2 协议，会经过防火墙，而且应用层有更丰富的控制。
- RESTful API 基于 XML 或 JSON 数据格式，便于阅读和调试。
- RPC 采用 TCP/UDP 协议，工作在第四层(传输层)，性能更高。
- RPC 基于 protocol buffer 二进制数据格式，传输更快。

