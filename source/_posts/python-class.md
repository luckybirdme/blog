---
title: python class
date: 2020-02-04 19:17:41
tags: python
---

> class

<!-- more -->

### 1. 创建 class

```python
class User():
	# 构造函数
	def __init__(self,name):
		self.name = name
	def show_name(self):
		print(self.name)
if __name__=='__main__':
	u = User('jack')
	u.show_name()
```

### 2. 继承父类

```python

class User():
	def __init__(self,name):
		self.name = name
	def show_name(self):
		print(self.name)

class Student(User):
	def __init__(self,name,grade):
		User.__init__(self,name)
		self.grade = grade
	def show_grade(self):
		print(self.grade)
if __name__=='__main__':
	u = Student('jack',10)
	u.show_name()
	u.show_grade()

```

### 3. 多个继承

```python

class User():
	def __init__(self,name):
		self.name = name
	def show_name(self):
		print(self.name)
class School():
	def __init__(self,addr):
		self.addr=addr
	def show_addr(self):
		print(self.addr)

class Student(User,School):
	def __init__(self,name,grade,addr):
		User.__init__(self,name)
		School.__init__(self,addr)
		self.grade = grade
	def show_grade(self):
		print(self.grade)
if __name__=='__main__':
	u = Student('jack',10,'shenzhen')
	u.show_name()
	u.show_grade()
	u.show_addr()
```


### 4. 重写父类方法

```python

class User():
	def __init__(self,name):
		self.name = name
	def show_name(self):
		print(self.name)

class Student(User):
	def __init__(self,name,grade):
		User.__init__(self,name)
		self.grade = grade
	def show_grade(self):
		print(self.grade)
	def show_name(self):
		print('rewrite %s' % self.name)
if __name__=='__main__':
	u = Student('jack',10)
	u.show_grade()
	u.show_name()
```

### 5. 私有属性和方法通过两个下横线标识

```python
class Student():
	def __init__(self,name,grade):
		self.name = name
		self.__grade = grade
	def show_grade(self):
		print(self.grade)
	def __get_name(self):
		print(self.name)
if __name__=='__main__':
	u = Student('jack',12)
	print(u.name)
	# 访问报错
	# print(u.__grade)
	# print(u.__get_name())
```


### 6. 默认情况下，实例化后的对象可随意增加属性，可通过 __slots__ 限制属性的增加

```python
# 随意增加
class User():
	__slots__ = ('name', 'age')
	def __init__(self):
		pass

u = User()
u.name = 'jack'
print(u.name)
u.grade = 10
print(u.grade)

# 属性限制
class User():
	__slots__ = ('name', 'age')
	def __init__(self):
		pass

u = User()
u.name = 'jack'
print(u.name)
# 增加会报错
# u.grade = 10
# print(u.grade)

```

### 7. 直接当做函数调用

```python

class Student():
	def __init__(self,name):
		self.name = name
	def __call__(self):
		print(self.name)
if __name__=='__main__':
	u = Student('jack')
	# 默认调用 __call__()
	u()
	u.__call__()
```


### 8. 通过 __enter__ 和 __exit__ 实现上下文管理

```python

# 通过 with 打开文件和自动关闭
with open('hello.txt','r') as f:
    print(f.readlines())

# 模拟 open 函数实现上下文管理
class myOpenFile:
    def __init__(self,filename,mode):
        self.filename=filename
        self.mode=mode
    def __enter__(self):
        print('open f')
        self.f = open(self.filename, self.mode)
        return self.f
    def __exit__(self, exc_type, exc_val, exc_tb):
        print('close f')
        self.f.close()

# 自动打开和关闭文件
with myOpenFile('hello.txt','r') as f:
    print(f.readlines())


```