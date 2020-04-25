---
title: python2 to 3
date: 2020-02-04 12:36:30
tags: python
---

> python2 had stopped at 2020
> python3 is future

<!-- more -->


### python3 解析器默认使用 utf-8 编码，python2 默认是 ASCII，如果有中文，需要指定 utf-8 编码

```python
# python2 default ascii
# -*- coding:utf-8 -*-
s='中文'


# python3 default utf-8
s='中文'

```

### python3 的 print 括号是必须的

```python
# 2
print('hello')
print 'hello'

# 3
print('hello')
# error
# print 'hello'

```


### python3 异常处理必须使用 as 关键字，python2 两种方式都支持
```python
# 2
try:
    1/0 
except Exception , e:
    print(e)

# 2 or 3
try:
    1/0 
except Exception  as e:
    print(e)

```

### python3 的 map 返回迭代器，filter 也一样

```python

#2 
# return list
li=map(lambda x: x+1, range(5))
print(isinstance(li,list))

# 3 
from collections.abc import Iterator
# return iterator
it=map(lambda x: x+1, range(5))
print(isinstance(it,Iterator))
```


### python2中有 range 和 xrange 两个方法。range 返回一个list，在被调用的时候即返回整个序列；xrange返回一个可迭代对象 Iterable，在每次循环中生成序列的下一个数字。python 3中不再支持 xrange 方法，Python 3中的 range 方法就相当于 Python 2中的 xrange 方法。
```python

# 2 
# return list
li=range(5)
print(isinstance(li,list))

# 3 
from collections.abc import Iterable
# return Iterable
it=range(5)
print(isinstance(it,Iterable))

```



### python3 整数相除返回浮点数
```python
# 2
# return 1
print('3 / 2 =', 3 / 2)

# return 1.5
print('3 / 2 =', 3 / 2)
```



### python3 不再支持 has_key

```python
p = {"age": 30}
# 2 
print("has key :  ", p.has_key("age"))

# 3
# error
print("has key :  ", p.has_key("age"))
# 使用 in 方式
print("has key :  ", "age" in p)
```

### python3 不再支持 <>，只保留 !=

```python
# 2
print(1!=2)
print(1 <> 2)


# 3
print(1!=2)

````


### python3 八进制必须使用 0o 开头
```python
# 2
print(0o0010)
# 显示 8
print(0010)


# 3
print(0o0010)
# error
print(0010)

```