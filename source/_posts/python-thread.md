---
title: python 进程 线程 协程
date: 2020-02-05 16:19:37
tags: python
---

> 进程是资源分配的基本单位，资源不共享，开销成本最高
> 线程是调度的基本单位，共享部分资源，开销成本中等
> 协程是一种轻量级线程，调度由用户控制，开销成本最低

<!-- more -->

## 一. 背景介绍

### 1. 名词解释
- 进程：一个运行的程序（代码）就是一个进程，没有运行的代码叫程序，进程是系统资源分配的最小单位，进程拥有自己独立的内存空间，所以进程间数据不共享，开销大，但是可以充分利用多个 CPU。
- 线程：调度执行的最小单位，不能独立存在，依赖进程存在一个进程至少有一个线程，叫主线程，而多个线程共享内存 (全局变量)，从而极大地提高了程序的运行效率。，但是一个进程的线程无法在多个 CPU 运行。
- 协程：是一种用户态的轻量级线程，不是被操作系统内核所管理，而是完全由用户程序控制调度。


### 2. 优缺点分析
#### 进程
- 优点
 - 进程之间资源独立，即使一个进程崩溃，对另外进程也不影响。
 - 在每个 CPU 上运行一个进程，合适计算机密集型任务

- 缺点
 - 需要维护多个进程之间的大量数据、栈堆等，开销成本高
 - 进程之间通信复杂，大数据传输也相对较慢

#### 线程
- 优点
 - 线程共享进程的部分资源，所以线程开销相对较低。
 - 线程可通过全局变量通信，合适大数据共享的任务。
- 缺点
 - 一个线程挂掉，会影响整个进程，稳定性不足
 - 线程之间存在资源抢占，需要建立锁机制，影响性能
 - 无法充分利用多个 CPU 的性能

#### 协程
- 优点
 - 在单线程，通过用户程序控制协程切换，切换成本比线程还低
 - 因为是用户控制，所以没有线程的锁机制，性能更高
 - 合适非 CPU 密集型的高并发任务，比如IO密集型
- 缺点
 - 包括了线程的缺点，参考上面
 - 如果一个协程阻塞，会阻塞整个线程


### 3. 多任务分类
- 计算密集型
计算密集型任务的特点是要进行大量的计算，消耗CPU资源，比如对视频处理等。因此，为充分利用 CPU 性能，启动进程数等于 CPU 数。这种完全依靠CPU性能的任务，对代码运行效率要求极高。Python 这样的脚本语言运行效率较低（边编译边执行），不太适合。对于计算密集型任务，最好用C语言编写。

- IO密集型
IO密集型，涉及到网络、磁盘IO的任务都是IO密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成，因为IO的速度远远低于CPU和内存的速度。所以代码运行效率影响不大，采用开发效率更高的脚本语言即可，比如 Python 。这种IO密集型任务，最好采用异步IO编程，非阻塞IO响应，比如 nodejs，可在单线程下达到高并发响应。



### 4. 多任务机制

- 并行和并发
并行是多个任务同一个时刻执行，并发是多个任务在同一个时间段执行，并不是同时执行。

- Python GIL
Global Interpreter Lock(全局解释器锁)，每个CPU在同一时间只能有一个线程获得的 GIL 锁，即只能执行一个线程

- 多任务实现
 - python 通过 multiprocessing 实现多进程，在多个 CPU 上运行。因为每个进程有各自独立的 GIL，互不干扰，所以可以充分利用 CPU 性能。
 - python 通过 threading 实现多线程，但是由于GIL 锁机制，在单个 CPU 下，只有一个线程获得 GIL 锁并执行，即使在多个CPU上运行多个线程，最终每个CPU也只有一个线程执行，python 的多线程只是伪线程，在单个CPU上分时间片执行（极短间隔），一个个线程排队执行。
 - 协程，又称微线程，英文名Coroutine。Python 多协程在单线程下运行，所以没有 GIL 锁的影响，而且通过用户程序进行子程序切换，效率更高。




## 二. Python 多进程实现

每个CPU单独运行一个 CPython 解释器，即每个CPU同时运行一个 Python 进程，所以 Python 多进程在多CPU下才会更高效，而且没有  GIL(Global Interpreter Lock) 锁的问题。

### 1. 创建子进程
```python
from multiprocessing import Process
import os

# 子进程要执行的代码
def run_proc(name):
    print('Child process start.')
    print('Run child process %s (%s)...' % (name, os.getpid()))
    print('Child process end.')

if __name__=='__main__':
    print('Parent process start %s.' % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print('Parent start Child process.')
    # 创建子进程
    p.start()
    # 等待子进程结束
    p.join()
    print('Parent process end %s.' % os.getpid())

```

### 2. 为了更高效地创建进程，通过进程池的方式，提前创建好，开箱即用

```python

from multiprocessing import Pool
import os, time

def long_time_task(name):
    print('Run task %s (%s) start' % (name, os.getpid()))
    start = time.time()
    time.sleep(1)
    end = time.time()
    print('Task %s runs %0.2f seconds end' % (name, (end - start)))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    # 创建进程池，共3
    p = Pool(3)
    for i in range(5):
        # 将线程保存到队列，等待执行
        print('save task %s to list' % i)
        p.apply_async(long_time_task, args=(i,))
    print('close pool')
    # 关闭线程池，不再接受线程
    p.close()
    # 开始执行线程，等待线程执行完毕
    print('Waiting for all subprocesses done...')
    p.join()
    print('All subprocesses done.')

```
### 3. 利用 with 和 ProcessPoolExecutor 可以更方便创建和关闭进程池，这些进程都是异步调用的，官网例子

```python
import concurrent.futures
import math

PRIMES = [
    112272535095293,
    112582705942171,
    112272535095293,
    115280095190773,
    115797848077099,
    1099726899285419]

def is_prime(n):
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False

    sqrt_n = int(math.floor(math.sqrt(n)))
    for i in range(3, sqrt_n + 1, 2):
        if n % i == 0:
            return False
    return True

def main():
    with concurrent.futures.ProcessPoolExecutor() as executor:
        # map(fn,*iterables) 合适 fn 是一样但参数不同，而且输出结果按照输入数组的顺序
        # submit(fn, *args, **kwargs) 合适 fn 不同，而且输出结果是无序的
        for number, prime in zip(PRIMES, executor.map(is_prime, PRIMES)):
            print('%d is prime: %s' % (number, prime))

if __name__ == '__main__':
    main()
```


### 4. 为了便于控制子进程的输入和输出，可通过 subprocess 创建子进程

```python
import subprocess

print('$ node')
# 创建子进程，执行 node 命令
p = subprocess.Popen(['node'], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
# 向子进程发送命令，并获取返回
stdout, stderr = p.communicate(b'console.log(\'hello\')\n.exit\n')
print(stdout.decode('utf-8',errors='ignore'))
print('Exit code:', p.returncode)

```


### 5. 进程可利用 Queue、Pipe 等多种方式通信

- Queue 

```python
from multiprocessing import Process, Queue
import os, time

# 写数据进程执行的代码:
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        # 把数据放到队列
        q.put(value)
        time.sleep(1)

# 读数据进程执行的代码:
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        # 等待队列数据
        value = q.get(True)
        print('Get %s from queue.' % value)

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    # 父进程创建Queue，并传给各个子进程：
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入:
    pw.start()
    # 启动子进程pr，读取:
    pr.start()
    # 等待pw结束:
    pw.join()
    # pr进程里是死循环，无法等待其结束，只能强行终止:
    pr.terminate()

```

- Pipe

```python
from multiprocessing import Process, Pipe
import os
 
def f(conn):
    print('Child process %s.' % os.getpid())
    # 向父进程发送
    print('Child process send msg')
    conn.send('hello')
    # 等待父进程信息
    print('Child process wait msg')
    msg = conn.recv()
    print('from parent: %s' % msg)
    conn.close()
 
if __name__ == '__main__':
    print('Parent process %s.' % os.getpid())
    parent_conn, child_conn = Pipe()
    p = Process(target=f, args=(child_conn,))
    p.start()
    print('Parent process wait msg')
    msg = parent_conn.recv()
    print('Parent from son: %s' % msg)
    print('Parent process send msg')
    parent_conn.send('world')
    print('Parent process wait Child end')
    p.join()
    print('Parent process end')

```



## 三. Python 多线程实现

在 CPython 编译器中， 因为 GIL(Global Interpreter Lock) 限制 CPU 每个时间片只能运行一个线程，所以实际 python 还是单线程排队执行，但是对于 IO 密集型任务来说，多线程还是有比较好性能优势的，比如爬虫抓取网页。

### 1. 简单的多线程
```python
import threading
import time

def thread_func(n):

    t = threading.currentThread()
    print("start threading %s" % n)
    #线程ID
    print('Thread id : %d' % t.ident)
    #线程NAME
    print('Thread name : %s' % t.getName())
    time.sleep(1)
    print("end threading %s" % n)

if __name__ == "__main__":
    print('Parent go')
    t1 = threading.Thread(target=thread_func, args=(1, ))     # 创建线程1
    t2 = threading.Thread(target=thread_func, args=(2, ))    # 创建线程2
    t1.start()    # 运行线程1
    t2.start()    # 运行线程2
    # 父进程等待子进程结束
    t1.join()
    t2.join()
    print("Parent over")
```

### 2. 通过 ThreadPoolExecutor 创建线程池更方便，官网例子如下
```python

import concurrent.futures
import urllib.request

URLS = ['http://www.foxnews.com/',
        'http://www.cnn.com/',
        'http://europe.wsj.com/',
        'http://www.bbc.co.uk/',
        'http://some-made-up-domain.com/']

# Retrieve a single page and report the URL and contents
def load_url(url, timeout):
    with urllib.request.urlopen(url, timeout=timeout) as conn:
        return conn.read()

# We can use a with statement to ensure threads are cleaned up promptly
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    # Start the load operations and mark each future with its URL
    future_to_url = {executor.submit(load_url, url, 60): url for url in URLS}
    # 返回一个生成器，等待每个线程执行完
    for future in concurrent.futures.as_completed(future_to_url):
        url = future_to_url[future]
        try:
            data = future.result()
        except Exception as exc:
            print('%r generated an exception: %s' % (url, exc))
        else:
            print('%r page is %d bytes' % (url, len(data)))
```


### 3. Global Interpreter Lock(全局解释器锁) 确保的是 CPython 编译器每时每刻只能有一个 thread 在执行 bytecode。如果一条语句有多个 bytecode，那么多个线程切换执行相同的 bytecode 时，可能会错乱。

比如 n = 0，然后执行 n = n+1，其实相当于执行三个 bytecode
1. 获取 n 值
2. 执行 n + 1
3. 返回给 n

以上三个步骤执行时，有可能发生线程切换，如果两个线程并发执行以上 bytecode，理论上 n 将会等于 2，但实际可能会发现以下情况

1. thread1 获取 n 值 (0) 
2. thread2 获取 n 值 (0) 
3. thread1 执行 n + 1 (1) 
4. thread2 执行 n + 1 (1) 
5. thread1 返回给 n (1) 
6. thread2 返回给 n (1) 

所以 threading 提供了一种 lock 机制，确保单个线程执行时，数据将会锁住，其他线程只能处于等待。

```python
import threading
import time

class myThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        print ("开启线程： " + self.name)
        # 获取锁，用于线程同步
        threadLock.acquire()
        print_time(self.name, 1, 3)
        # 释放锁，开启下一个线程
        threadLock.release()

def print_time(threadName, delay, counter):
    while counter:
        time.sleep(delay)
        print ("%s: %s" % (threadName, time.ctime(time.time())))
        counter -= 1

threadLock = threading.Lock()
threads = []

# 创建新线程
for x in range(1,5):
    threads.append(myThread(x,"Thread-%s" % x,x))

# 开启新线程
for t in threads:
    t.start()

# 等待所有线程完成
for t in threads:
    t.join()
print ("退出主线程")

```

以上机制虽然能保护数据安全，但是也会影响性能，所以 Python 多线程其实效率不大好
为了更高的性能，这时候就轮到协程上场了

## 四. Python 协程实现

协程本质是函数回调，在单线程下，通过用户控制子程序切换，不需要 os 调度，不需要考虑抢占锁，效率极高。
但协程也有缺点，由于需要用户调度，程序编写相对复杂，而且如果一个子程序卡住，会堵塞整个线程，稳定性不好。

### 1. yield 关键字，用于返回生成器 generator 函数，只能用于函数内部，函数执行到 yield 会自动停止，返回到主程序，等待下次主程序调用生成器时，再继续执行。

#### 使用场景：生成的正整数 list，如果一次性生成，将会耗费大量内存，通过生成器则可避免此问题

```python
def createIntList():
    for x in range(1,10000):
        yield x
    # 简写方式
    # yield from range(1,10000)

for n in createIntList():
    print(n)
    if n > 10:
        break
```
#### 使用场景：利用 yield 暂停子程序，返回主程序的特点，在用户态快速切换主和子程序，可模拟协程的使用。

```python
# 模拟协程
# 生成器函数
def consumer():
    r = 'init'
    print('[CONSUMER] start')
    while True:
        print('[CONSUMER] while')
        # 等待主程序调用执行，执行完后，再次阻塞等待下次调用
        # r 是协程返回给主程序的参数，n 是主程序返回给协程的参数
        n = yield r
        print('[CONSUMER] get %s' % n)
        if not n:
            print('[CONSUMER] return')
            return
        print('[CONSUMER] Consuming %s' % n)
        r = 'OK %s ' % n
    print('[CONSUMER] end')
# 主程序
def produce(c):
    print('[PRODUCER] start')
    # 初始化生成器
    r = c.send(None)
    print('[PRODUCER] init [CONSUMER] get %s' % r)
    n = 0
    while n < 5:
        n = n + 1
        # 产生任务 n
        print('[PRODUCER] Producing %s' % n)
        # 传递任务 n 给协程处理，并获得处理后的返回 r
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

c = consumer()
produce(c)

```



### 2. python3.4 引入 asyncio 作为的异步 IO 标准库，实现事件循环（Event Loop），主程序提交任务后无需等待子程序执行完毕，可以继续执行其他事物，比如继续执行其他协程。

```python

import threading
import asyncio

# 装饰器，在函数执行前后，做一些特殊处理
'''
def asyncio.coroutine(hello):
    def wrapper(*args, **kw):
        print('call start %s():' % func.__name__)
        r = func(*args, **kw)
        print('call end %s():' % func.__name__)
        return r
    return wrapper
'''
@asyncio.coroutine
def hello(n):
    t = threading.currentThread()
    print('%s task start %s' % (n,t.ident))
    # 模拟异步任务执行，等待 1 秒
    yield from asyncio.sleep(1)
    print('%s task end %s' % (n,t.ident))


print('start')
# 获取事件循环
loop = asyncio.get_event_loop()
tasks=[]
# 创建多个协程任务
print('list tasks')
for x in range(1,5):
    tasks.append(hello(x))
# 将任务放到事件循环中执行
# 因为是异步执行，无需等待执行完毕，可以一次性把所有协程都提交
print('exec tasks')
async_task = asyncio.wait(tasks)
# 等待所有协程执行完毕
print('wait tasks')
loop.run_until_complete(async_task)
# 关闭事件循环
print('loop close')
loop.close()

print('end')

```


### 3. python3.5 增加 async/await ，在语言上原生支持异步编程，代码更简洁，python3.8 已经提倡使用此方式来代替 yield，避免跟生成器语法混淆。

```python
import threading
import asyncio

async def hello(n):
    # 确认在同一个线程内执行
    t = threading.currentThread()
    print('%s task start %s' % (n,t.ident))
    # 模拟异步任务执行，等待 1 秒
    # await 后面必须是 awaitable 对象，即实现了 __await__ 方法的 Object
    await asyncio.sleep(1)
    print('%s task end %s' % (n,t.ident))

async def main():
    # 将所有协程放到 task 里
    await asyncio.gather(*[hello(x) for x in range(1,5)])


print('start')
# 启动一个事件循环，执行所有 task，即协程
# 自动关闭事件循环
asyncio.run(main())

print('end')

```