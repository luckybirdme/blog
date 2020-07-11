---
title: python 知识点
date: 2020-04-24 19:59:06
tags: Python
---

> 基于 Python 3.8

<!-- more -->

### 序列化和反序列化作用
- 将对象从一个地方传递到另一个地方，比如JSON数据传输，这是以字符串形式的进行转换

```python
# JSON 格式数据转换
# 对象转 JSON 字符串 
obj=['foo', {'bar': ('baz', None, 1.0, 2)}]
out=json.dumps(obj)
print(out)
# JSON 字符在转对象
new=json.loads(out)
print(new)
print(type(new))

```
- 以某种存储形式持久化对象，比如将列表数据保存到文件，可采用 pickle，以二进制方式序列化，但是 unpickle 数据存在安全问题，官方推荐使用 JSON

```python
# 将对象 obj 封存以后的对象写入已打开的 file object file。
pickle.dump(obj, file)
# 从已打开的 file 文件 中读取封存后的对象，重建其中特定对象的层次结构并返回。
obj=pickle.load(file)
```

### Python 切片使用

```python
l=[1,2,3,4,5,6]
print(l[1:])
print(l[3:5])

```

### Python 字典合并

```python
d1={'key':'val','key1':'val1'}
d2={'key1':'val2','key2':'val2','key3':'val2'}
d3={'key1':'val3','key2':'val3','key3':'val3'}
d2.update(d1)
print(d2)
d1.update(d3)
print(d1)
```

### Python 列表去重

```python

l=[1,2,3,4,2,3]
s=set(l)
print([x for x in s])

```

### Python new 和 init 方法区别
- new 调用以创建一个 cls 类的新实例，如果 return 实例，则会自动调用 init 方法，否则不会调用
- init 将在类实例化后并且 new 返回实例时才会执行，其中 self 参数是 new 返回的实例

```python
# new 先于 init 执行
class TestNewAndInit():
    def __new__(cls,*args,**kwargs):
        print('__new__')
        # 返回实例
        return super(TestNewAndInit, cls).__new__(cls)
    def __init__(self,name):
        self.name = name
        print('__init__')
    def show_name(self):
        print(self.name)

test_my_class_one= TestNewAndInit('test one')
test_my_class_two= TestNewAndInit('test two')
test_my_class_one.show_name()
test_my_class_two.show_name()
print(test_my_class_one)
print(test_my_class_two)


# 通过 new 实现单例模式
class TestSingleton():
    __instance = None
    def __new__(cls, *args, **kwargs):
        if cls.__instance is None:
            cls.__instance = super(TestSingleton, cls).__new__(cls)
        return cls.__instance
    def __init__(self,name):
        self.name = name
        print('__init__')
    def show_name(self):
        print(self.name)
test_singleton_one= TestSingleton('test one')
test_singleton_two= TestSingleton('test two')
test_singleton_one.show_name()
test_singleton_two.show_name()
print(test_singleton_one)
print(test_singleton_two)
# test two
# test two
# <__main__.TestSingleton object at 0x00000000022D25E0>
# <__main__.TestSingleton object at 0x00000000022D25E0>


```

### 列表[1,2,3,4,5],请使用map()函数输出[1,4,9,16,25]，并使用列表推导式提取出大于10的数，最终输出[16,25]

```python

l=[1,2,3,4,5]
ll=list(map(lambda x : x**2 ,l))
print(ll)
lll=[x for x in ll if x > 10]
print(lll)

```

### Python 通过 random 返回伪随机数， 不应将此模块的伪随机生成器用于安全目的。

```python
import random

# 返回 [0.0, 1.0) 范围内的下一个随机浮点数。
print(random.random())
print(random.random())

# 返回指定返回的整数，a <= N <= b
print(random.randint(1,10))
print(random.randint(20,100))
```

### Python 字符串 "ajldjlajfdljfddd" 去重，并从大到小排序输出

```python
l = "ajldjlajfdljfddd"
s = set(l)
ll = list(s)
print(ll)
ll.sort(reverse=True)
print(ll)
```

### list.sort 和 sorted
- list.sort() 函数只适用于列表排序，sorted() 函数适用于任意可以迭代的对象排序，所以 sorted() 应用范围更广。
- list.sort() 函数排序会改变原有的待排序列表，而 sorted() 函数则不会改变，只是返回经过排序的新对象。如果无需保存原列表，则优先使用 list.sort() 节省内存空间。

```python

l=[2,3,4,1]
print(l)
l.sort(reverse=True)
# 原理的list被改变了
print(l)
ll=[2,3,4,1]
print(ll)
# 直接返回新的对象
lll = sorted(ll,reverse=True)
print(lll)

```

### 替换空格

```python

a="h e l l o"
print(a)
print(a.replace(" ",""))
print(a)
print("".join(a.split(" ")))

```

### Python if 原理是通过 bool 函数来判断真假，以下值会被解析为假值: False, None, 数字零，以及空字符串和空容器（包括字符串、元组、列表、字典、集合）。 所有其他值都会被解析为真值

```python
print(bool(False))
print(bool(None))
print(bool(0))
print(bool(''))
print(bool([]))
print(bool({}))
print(bool(()))
```

### 单下划线、双下划线、头尾双下划线说明：
- \_\_foo\_\_: 定义的是特殊方法，一般是系统定义名字 ，类似 \_\_init\_\_() 之类的。
- \_foo: 以单下划线开头的表示的是 protected 类型的变量，即保护类型只能允许其本身与子类进行访问，不能用于 from module import *
- \_\_foo: 双下划线的表示的是私有类型(private)的变量, 只能是允许这个类本身进行访问了。



### 函数传参数是传值还是传址

- 对于不可变类型（数值型、字符串、元组），因变量不能修改，所以运算不会影响到变量自身，此时传递是值
- 而对于可变类型（列表字典）来说，函数体运算可能会更改传入的参数变量，所以此时传递是变量引用

```python
def edit(param):
    if 'key' in param:
        param['key']='edit obj'
        print('obj',param)
    else:
        param = 'edit str'
        print('str',param)
# 可变对象
obj={'key':'val'}
# 不可对象
sss='123'
print(obj)
edit(obj)
# 已经被改变
print(obj)

print(sss)
edit(sss)
# 不会变
print(sss)
```


### Python 代码样式规范 PEP8
- 行缩进 4 个空格，尽量不用使用 tab

```python
if True:
    # 缩进 4 个空格
    print('hello')

```
- 代码文件编码使用 UTF-8，Python2默认ASCII，需要声明 UTF-8
- Python3默认UTF-8，所以不用编码声明了

```python
# -*- coding:utf-8 -*-
# python 2 code
```

- 变量命名：应该小写，多个单词用下划线隔开

```python
school_name='school_name'

```

- 常量名：应该全部大写，多个单词用下划线隔开

```python
MAX_LINE=100

```

- 函数命名：应该小写，多个单词用下滑线分割

```python
def show_name(name):
    print(name)

```

- 类命名：一般使用首字母大写的约定，使用驼峰式(CamelCase)

```python
class TestCounter():
    def test_main():
        num = 1
```
- 模块名：尽量使用小写命名，多个单词用下横线

```python
import my_lib
from my_lib import SendEmail
```


### open 文件的模式

模式  |描述
-|-
r   |以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。
rb  |以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。
r+  |打开一个文件用于读写。文件指针将会放在文件的开头。
rb+ |以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。
w   |打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
wb  |以二进制格式打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
w+  |打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
wb+ |以二进制格式打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
a   |打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
ab  |以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
a+  |打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
ab+ |以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。

### Python 语言执行原理
- Python 语言是一种解释语言，源代码经过解释器成字节码后才能被机器执行，目前主流解释器是 CPython。
- Python 源代码解释后的文件后缀名是 pyc，类似 Java 解释后的 class 文件，可以被虚拟机 JVM 执行。


### Python 内存管理机制
- 引用计数机制，记录对象引用次数，当次数等于 0，则标记清理
- 垃圾回收机制，对于标记清理的对象，将会注销这些对象。
- 内存池机制，它将不用的内存放到内存池，而不是返回给操作系统。


