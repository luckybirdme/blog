---
title: python list tuple dict set
date: 2020-02-03 09:38:21
tags: python
---

> list ，列表，是一种有序的集合，可以随时添加和删除其中的元素，存放不同数据类型。
> tuple ，元组，是另一种有序的列表，与 list 非常类似，但是创建完毕后就不能修改了。
> dict ，字典，是一种 key-value 保存的无序集合，查询速度快，消耗内存也多。
> set ，集合，是一种去重式的 key 无序集合，类似没有 value 的 dict 。

<!-- more -->

## 一. 数据类型介绍

### 1. list 是一种有序的集合，可以随时添加和删除其中的元素，存放不同数据类型，而且允许重复的值，通过 [] 来表示

- 例子

```python
num = [1,2,3]
obj = ['a','a', 1 , { 'key': 'val'}]
```

- 增删改查

```python
num = [1,2,3]
num.append(4)
print(num)
num.insert(2,22)
print(num)
num[0]=0
print(num)
del num[1]
print(num)
num.remove(22)
print(num)
```


### 2. tuple 可以看做是一种“不变”的 list，访问也是通过下标，但创建后就不能修改，用小括号 () 表示： 

- 例子

```python
obj = ('1','a','1',{'key':'val'})
# t = (1) 会表示成 tuple 只有一个元素
# 多个逗号，避免歧义，表示 tuple 里有一个元素，值为 1
num = (1,)
print(num)
print(obj)

```

- 虽然不能修改元组的元素，但可以合并元祖
```python
one=(1,3)
two=(2,4)
m=one+two
print(m)
```

- tuple 不能修改，所以就没有 append insert pop 的方法，但是对于引用型数据(类似 dict { key : val })，下标不变的情况下，具体指向内容是可变的。

```python
obj = ('1','a','1',{'key':'val'})
print(obj)
print(obj[1])
# 不能修改，报错 TypeError: 'tuple' object does not support item assignment
# obj[1] = 'b'
# 可正常修改
obj[3]['key'] = 'test'
print(obj)
```

### 3. dict 是一种 { key : val } 保存的无序集合，key 不能重复，底层通过 HashMap 生成高效的查找树(类似红黑树)，查询速度不会因为数据量增大而出现明显的性能下降，而 list 的查找速度随着元素增加而逐渐下降。不过 dict 的查找速度快不是没有代价的，缺点是占用内存大，还会浪费很多内容，list 正好相反，占用内存小，但是查找速度慢。


- 例子

```python

d = { 'one':'a', 'two':'b'}
print(d)

```

- 增删改查

```python
d = { 'one':'a', 'two':'b'}
print(d)
print(d['two'])
# key 不存在会报错
# d['not'] = 'aa'
# 修改
d['one'] = 'aa'
d['three'] = 'cc'
d.pop('two')
print(d)
del d['one']
print(d)
# 如果 key 不存在，可指定默认值
print(d.get('two','not exist'))
```

### 4. set 是一种去重式的 key 无序集合，类似没有 value 的 dict ，无序不重复，所以也无法通过索引访问，只能判断是否存在某个 key。

- 例子

```python
# 通过 list 来创建
s1 = set([1,2,'3'])
print(s1)
# 使用大括号
s2 = {1,2,'3'}
print(s2)

```

- 增删改查

```python
s = {1,2,'3',3}
print(s)
print (1 in s)
print ('2' in s)
print ('3' in s)
# 增加
s.add(5)
print(s)
# 删除
s.remove('3')
print(s)

```


## 二. 区别和用途

### 1. 区别

数据类型|list|tuple|dict|set 
-|-|-|-|-
创建|[1,2,1,{'key':'val'}]|(1,2,1,{'key':'val'})|{ 'key1' : 1, 'key2' : 2 }|set([1,2])
有序|是|是|否|否
重复|是|是|否|否
访问|l[0]|t[0]|d['key'],d.get('key')|只能判断存在 print 1 in s
修改|l[0]='a'|无法修改结构，但可修改引用类型的具体内容，t[3]['key']='b'| d['key1']='a'|无法修改
增加|append,insert|无|d['key3'] = 'val'|s.add(3)
删除|pop,pop(2)|无|d.pop('key3')|s.remove(1)


### 2. 用途
#### list
- 存放有序数组，合适多种数据类型
- 随着元素增多，查询耗时明显增加，但是占用内存小

#### tuple
- 常量数组，防止被修改
- 函数返回其实就是一个 tuple

```python
def fn():
	return 1,2
x,y = fn()
print(fn())
```

#### dict
- 快速操作数据的字典，在大量元素下依然性能可观
- 消耗更多内存，用空间换取时间

#### set
- 无序不重复元素集，可以计算交集、差集和并集等

```python
a = set('abc')
b = set('bcd')
print(a&b)
one=set([1,2,3])
two=set([2,3,4])
print(one|two)

```
