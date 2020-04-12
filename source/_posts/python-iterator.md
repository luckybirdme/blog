---
title: python Iterable Iterator
date: 2020-02-03 21:39:43
tags: python
---

> Iterable, Iterator

<!-- more -->


### 1. 可以直接作用于for循环的对象统称为可迭代对象 : Iterable，包括 list、tuple、dict、set、str，以及生成器和带yield的generator function 
```python
from collections.abc import Iterable

print(isinstance([1, 2, 3, 4, 5], Iterable))
print(isinstance('abc', Iterable))
for x in [1, 2, 3, 4, 5]:
    print (x)


```

### 2. 可以被next()函数调用并不断返回下一个值的对象称为迭代器：Iterator，通过iter()函数获得一个Iterator

```python
from collections.abc import Iterator

print(isinstance(iter([1, 2, 3, 4, 5]), Iterator))
print(isinstance(iter('abc'), Iterator))

# 模拟 for 循环
def myFor(li,cb):
    # 转换成迭代器
    it = iter(li)
    while True:
        try:
            # 获得下一个值:
            x = next(it)
            cb(x)
        except StopIteration:
            # 遇到StopIteration就退出循环
            break
def myCb(x):
    print(x)

li=[1, 2, 3, 4, 5]
myFor(li,myCb)
```