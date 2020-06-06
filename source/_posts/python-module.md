---
title: python 引入模块 包 class
date: 2020-02-04 18:01:37
tags: Python
---

> from import

<!-- more -->

### 1. 引用模块

```python
# cat.py
def show_name():
	print('I am cat')

def show_age():
	print('My age is 10')

# main.py
# 引入整个文件内容
import cat
# 引入文件中具体方法
from cat import show_age
cat.show_name()
show_age()

```


### 2. 引入模块包

```python
# 模块包目录结构
# - pkg
#   - __init__.py
#   - cat.py
#   - dog.py

# main.py
# 引入指定模块名
import pkg.cat
import pkg.dog
pkg.cat.show_name()
pkg.dog.show_age()

```

- 为了避免每次重复写目录名，可这样引入

```python
# main.py
from pkg import cat,dog
cat.show_name()
dog.show_age()
```

- 也可以借助 __init__.py 文件一次性引入，但是不推荐这样，因为需要在 __init__.py 添加参数 __all__

```python
# __init__.py
__all__ = ["cat","dog"]

# main.py
from pkg import *
cat.show_name()
dog.show_age()

```


### 3. 引入 class

```python
# user.py
class User():
	def __init__(self,name):
		self.name = name
	def show_name(self):
		print(self.name)


# main.py
import user
u = user.User('luse')
u.show_name()

# 或者
# main.py
from user import User
u = User('jack')
u.show_name()

```


### 4. 为了方便调试，文件最后可增加 __name__=='\_\_main\_\_'，用以 python 命令直接执行来测试

```python
# user.py
class User():
	def __init__(self,name):
		self.name = name
	def show_name(self):
		print(self.name)

if __name__=='__main__':
	u = User('test')
	u.show_name()

# python user.py
```