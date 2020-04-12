---
title: python 函数参数
date: 2020-02-03 18:21:15
tags: python
---
> \*args
> \*\*kw

<!-- more -->

### 1. 普通参数
```python

def fn(a):
    print(a)

fn('a')

# 使用关键字，可以不按照顺序
def fn2( name, age ):
   print (name)
   print (age)
   return
fn2('Lose', 16)
fn2(age=18, name="Jack")

```


### 2. 可选参数
```python

def fn(a, b='test'):
    print(a)
    print(b)

fn('a')
fn('a','b')

```


### 3. 可变参数，args 接收到的是一个 tuple

```python
def fn(a, b='test', *args):
    print(a)
    print(b)
    print(args)

fn('a')
fn('a','b')
fn('a','b',1,2)
num=[3,4]
fn('a','b',*num)
num=(5,6)
fn('a','b',*num)

```


### 4. 关键字参数，kw 接收到的是一个 dict

```python
def fn(a, b='test', **kw):
    print(a)
    print(b)
    print(kw)

fn('a')
fn('a','b')
fn('a','b', aa='11',bb='22')
d={ 'one':'1','two':'2' }
fn('a','b', **d)
```