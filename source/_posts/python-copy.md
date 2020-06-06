---
title: python copy
date: 2020-04-23 18:29:04
tags: Python
---

> copy

<!-- more -->


1. 对于非引用数据类型，直接保存的就是数据，复制的也是数据，等号赋值相当于复制。
```python
# 非引用数据类型，直接保存的就是数据，复制的也是数据
aa='aa'
bb=aa
print('aa',aa)
print('bb',bb)
bb='bb'
print('aa',aa)
print('bb',bb)
```

2. 对于引用数据类型，复制包括引用，以及引用指向的数据，等号赋值只是复制了引用，称为浅拷贝
```python
# 引用数据类型，这里复制了引用，所以复制后的数据改变，会影响原有数据
a=[{'key1':'val1'}]
b=a
print('a',a)
print('b',b)
b[0]['key1']='b_val'
print('a',a)
print('b',b)
```

3. 深拷贝不仅复制引用，还把引用指向的数据也复制了
```python
import copy


c=copy.copy(a)
d=copy.deepcopy(a)
print('a',a)
print('b',b)
print('c',c)
print('d',d)
b[0]['key1']='b_val'
c[0]['key1']='c_val'
d[0]['key1']='d_val'
# 浅克隆，只复制了引用
# b 改变了，a 也改变
print('a',a)
print('b',b)
print('c',c)
# 深克隆，复制了引用，以及引用指向的数据
# d 改变了，a b c 都没变
print('d',d)
```