---
title: python generator
date: 2020-02-03 21:39:40
tags: python
---

> 直接创建一个列表，受到内存限制，列表容量肯定是有限，而且100万个元素的列表，非常占用内存。
> 列表元素可以按照某种算法推算出来，需要的时候才生成，那么就能解决上面的问题，这个由生成器 generator 实现。

<!-- more -->


### 1. 列表和生成器 generator
```python
l = [x * x for x in range(10)]
g = (x * x for x in range(10))
print(l)
print(g)
```

### 2. generator 通过 next() 生成具体元素

```python
g = (x * x for x in range(10))
print(next(g))
print(next(g))
print(next(g))
```

### 3. generator 是可迭代对象，可使用 for 访问
```python
g = (x * x for x in range(10))
for i in g:
	print(i)
```

### 4. 通过 yield 定义生成器 generator，在每次调用 next() 的时候执行，遇到 yield 语句会返回，再次执行 next() 时,从上次返回的 yield 语句处后面继续执行，最后抛出 StopIteration 错误信息

```python

def fn():
    for x in range(10):
        yield x * x
    return False
g = fn()
while True:
    try:
        x = next(g)
        print('g:', x)
    except StopIteration as e:
        print('Generator return value:', e.value)
        break
for i in g:
    print(i)
```


### 5. generator 生成无穷尽的列表，比如[杨辉三角形](https://baike.baidu.com/item/%E6%9D%A8%E8%BE%89%E4%B8%89%E8%A7%92)

```python
def triangles():
    g = []
    while True:
        gl = len(g) + 1
        l = []
        if gl == 1:
            pass
        elif gl == 2:
            l.append(1)
        else:
            y=g[-1]
            l.append(1)
            for x in range(gl-2):
                l.append(y[x]+y[x+1])
        l.append(1)
        g.append(l)
        yield l
n = 0
results = []
for t in triangles():
    results.append(t)
    n = n + 1
    if n == 10:
        break

for t in results:
    print(t)

```