---
title: python class
date: 2020-02-04 19:17:41
tags: Python
---

> class

<!-- more -->

### 1. 创建 class，实例化不需要 new

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

### 2. 继承父类，当做参数传递

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

### 3. 多个继承，传递多个父类参数

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


### 6. 默认情况下，实例化后的对象可随意增加属性，可通过 \_\_slots\_\_ 限制属性的增加

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

### 7. 通过 \_\_call\_\_ 直接将类当做函数调用

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


### 8. 通过 \_\_enter\_\_ 和 \_\_exit\_\_ 实现上下文管理

```python

# 通过 with 打开文件和自动关闭
with open('hello.txt','r') as f:
    print(f.readlines())

# 模拟 open 函数实现上下文管理
class myOpenFile:
    def __init__(self,filename,mode):
        self.filename=filename
        self.mode=mode
    # with 默认会调用
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

### 9. 通过 \_\_iter\_\_ 或者 \_\_getitem\_\_ 变成迭代对象

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

### 10. 初始化类时，调用 \_\_init\_\_，删除对象时，调用 \_\_del\_\_

```python

class TestMyClass():
    def __init__(self,name):
        print('init')
        self.name = name
    def __del__(self):
        print('del')
    def show_name(self):
        print(self.name)

test_my_class = TestMyClass('test')
test_my_class.show_name()
test_my_class = None
```

### 11. 访问不存在的对象成员属性的时候自动触发 \_\_getattr\_\_

```python

class TestMyClass():
    def __init__(self,name):
        print('init')
        self.name = name
    def show_name(self):
        print(self.name)
    def __getattr__(self,attr):
        print('__getattr__')
        print(attr,'not exist')
test_my_class = TestMyClass('test')
test_my_class.show_name()
print(test_my_class.show_age())
```

### 12. 添加或者修改对象成员的时候自动触发 \_\_setattr\_\_

```python
class TestMyClass():
    def __init__(self,name):
        print('init')
        self.name = name
    def show_name(self):
        print(self.name)
    def __setattr__(self,attr,value):
        print('__setattr__')
        print(attr,' set to ',value)
        # 注意此处不可以用self.key = value
        # 否则引起递归次数过大的错误
        # RecursionError: maximum recursion depth exceeded while calling a Python object
        super().__setattr__(attr,value) 
test_my_class = TestMyClass('test')
test_my_class.show_name()
test_my_class.name = 'hello'
test_my_class.show_name()

```