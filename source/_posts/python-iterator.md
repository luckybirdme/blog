---
title: python Iterator Iterable generator
date: 2020-02-03 21:39:43
tags: python
---

> Iterator, Iterable, generator

<!-- more -->

## 一. 迭代器(Iterator)
#### 1. 定义
实现了迭代器协议，该协议是指在容器内进行迭代，产生一系列值。
#### 2. 实现：同时实现以下两个方法
- iterator.\_\_next\_\_()
从容器中返回下一项。 如果已经没有项可返回，则会引发 StopIteration 异常

- iterator.\_\_iter\_\_()
返回迭代器本身。 这方法允许迭代器配合 for 语句使用，for 语句通过 \_\_iter\_\_() 方法获取迭代器，然后循环调用它的 \_\_next\_\_()方法，以便迭代获取容器的所有值。如果捕捉到 StopIteration 异常，则自动停止。


```python
#!/usr/bin/env python3
# 实现迭代器
class MyIterator():
    def __init__(self, text):
        print('init')
        self.text = text
        self.index = -1
    def __next__(self):
        try:
            self.index += 1
            return self.text[self.index]
        except IndexError:
            raise StopIteration()
    def __iter__(self):
        return self

# 获取迭代器
test_MyIterator_one = MyIterator('hello world one')

# 模拟 for 循环
def myFor(input_iterator):
    # 获取
    while True:
        try:
            # 获得下一个值:
            x = input_iterator.__next__()
            print(x)
        except StopIteration:
            # 遇到 StopIteration 就退出循环
            break
myFor(test_MyIterator_one)

# 以上相当于 for in
# 由于迭代器只能循环一次，所以只会输出一次结果，所以必须重新获取
test_MyIterator_two = MyIterator('hello world two')
for i in test_MyIterator_two:
      print(i)

test_MyIterator_three = MyIterator('hello world three')
# 除了 for in, 还可以通过 next 方法调用迭代器，但是需要手动捕捉 StopIteration
while True:
    try:
        x = next(test_MyIterator_three)
        print(x)
    except StopIteration:
        # 遇到 StopIteration 就退出循环
        print('StopIteration')
        break
```


## 二. 迭代对象(Iterable)
#### 1. 定义
能够逐一返回其成员项的对象，实质是利用迭代器来实现，可迭代对象的例子包括 list、tuple、dict、set、str 等

#### 2. 实现：只需其中以下一个方法
- object.\_\_iter\_\_() 方法返回迭代器，默认首先调用此方法
- object.\_\_getitem\_\_() 方法返回 self[key] 的求值，for 语句调用此方法循环时，会捕捉 IndexError 异常

```python
class MyIterableOne():
    def __init__(self, text):
        print('init')
        self.text = text
    def __iter__(self):
        # 创建迭代器，并返回
        return iter(self.text)

class MyIterableTwo():
    def __init__(self, text):
        print('init')
        self.text = text
        self.index = -1
    def __getitem__(self):
        self.index += 1
        return self.text[self.index]


test_MyIterableOne = MyIterableOne('hello world one')
test_MyIterableTwo = MyIterableTwo('hello world two')

for i in test_MyIterableOne:
    print(i)
for i in test_MyIterableTwo:
    print(i)
```


## 三. 生成器(generator)
#### 1. 定义
- 返回一个 generator iterator 的函数。它看起来很像普通函数，不同点在于其包含 yield 表达式以便产生一系列值供给 for 循环使用或是通过 next() 函数逐一获取
- generator iterator 函数执行时，遇到每个 yield 都会临时暂停处理，记住当前位置执行状态，包括局部变量和挂起的 try 语句。当该 generator iterator 函数恢复时，它会从离开位置继续执行。
- for 或者 next 方式调用生成器，传递参数是 None，通过 send() 方式才能传递参数给生成器，首次调用 send() 必需先传递 None

#### 2. 实现
- 通过关键字 yield，在函数中使用 yield 实现，相当于 return，但只是挂起函数，等待下次执行。
- 通过生成器表达式， (expression for var in iterable)，即可返回生成器

```python

#!/usr/bin/env python3
# 生成器函数 - 斐波那契
def fibonacci(n): 
    a, b, counter = 0, 1, 0
    while True:
        if (counter > n):
            print('stop counter %s' % counter) 
            return
        # 每次执行到这里就暂停，并返回 a
        # s 是主函数传递给生成器的参数
        s = yield a
        print('s : %s ' % s)
        a, b = b, a + b
        counter += 1

# f 是一个迭代器，由生成器返回生成
f = fibonacci(10) 

# 通过 for 方式调用
for x in f:
    print(x)

# 迭代器只能执行一次，所以重新获取
ff = fibonacci(10) 
# 传递参数给生成器
# 首次调用生成器
a = ff.send(None)
while True:
    try:
        print(a)
        a = ff.send('send')
    except StopIteration:
        print('stop for main')
        break

# 生成器表达式
fff = (x**2 for x in [1,2,3,4,5])
print(fff)
```






## 四. 关联
#### 1. 迭代器实现了\_\_iter\_\_()方法，所以迭代器也是迭代对象
#### 2. 生成器返回就是一个迭代器，所以生成器也是可迭代对象，可通过小括号将可迭代对象转为生成器
```python

l=[i for i in range(3)]
g=(i for i in range(3))


```
#### 3. 生成器使用场景
- 每次需要数据时才生成，不必一次性生成大量数据
- 实现了迭代器，可通过 for 或者 next 执行迭代

具体案例：杨辉三角形

![](/img/2020/triangle.png)

```python

def triangles():
    cur_list = []
    while True:
        cur_list_len = len(cur_list)
        if cur_list_len == 0:
            cur_list=[1]
        else:
            cur_list.insert(0,1)
            for i in range(1,cur_list_len):
                cur_list[i]=cur_list[i]+cur_list[i+1]
        yield cur_list

n = 5
for l in triangles():
    print(l)
    if n <= 0:
        break
    n -= 1
```