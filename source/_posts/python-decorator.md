---
title: python decorator
date: 2020-03-19 07:54:21
tags: Python
---
> python decorator

<!-- more -->

### 1. 普通函数
```python

def main():
    print('start : %s' % main.__name__)
    print('do something : %s' % main.__name__)
    print('end : %s' % main.__name__)

# 执行函数
main()

```


### 2. 想在函数执行前后打印日志
```python
print('log before main')
main()
print('log afrer main')
```


### 3. 上面添加日志的方法不太友好，使用高级编程技巧：装饰器
```python
def log(func):
    def wrapper():
        print('log before func : %s' % func.__name__)
        print('call func : %s' % func.__name__)
        result = func()
        print('log after func : %s' % func.__name__)
        return result
    return wrapper
def main():
    print('start : %s' % main.__name__)
    print('do something : %s' % main.__name__)
    print('end : %s' % main.__name__)

# 初始化装饰器
wrapper = log(main)
# 执行函数
wrapper()
```

### 4. 通过语法糖来调用装饰器
```python
# 装饰器的语法糖
@log
def main():
    print('start : %s' % main.__name__)
    print('do something : %s' % main.__name__)
    print('end : %s' % main.__name__)

# 执行函数
main()

```


### 5. 上面的main函数\__name__被改成wrapper，需要保留函数的元信息
```python
import functools

def log(func):
    # 传递 func 函数的元信息
    @functools.wraps(func)
    def wrapper():
        print('log before func : %s' % func.__name__)
        print('call func : %s' % func.__name__)
        result = func()
        print('log after func : %s' % func.__name__)
        return result
    return wrapper

@log
def main():
    print('start : %s' % main.__name__)
    print('do something : %s' % main.__name__)
    print('end : %s' % main.__name__)

main()

```


### 6. main 函数需要传递参数
```python


import functools
def log(func):
    # 传递 func 函数的元信息
    @functools.wraps(func)
    # 传递 func 的参数
    def wrapper(*args, **kwargs):
        print('log before func : %s' % func.__name__)
        print('call func : %s' % func.__name__)
        print('func args : {}'.format(*args))
        # 传递 func 的参数
        result = func(*args, **kwargs)
        print('log after func : %s' % func.__name__)
        return result
    return wrapper

@log
def main(p):
    print('start : %s' % main.__name__)
    print('input is : %s' % p)
    print('do something : %s' % main.__name__)
    print('end : %s' % main.__name__)

main('hello')

```

### 7. 装饰器本身也可支持参数
```python
import functools

def log(p):
    def decorator(func):
        # 传递 func 函数的元信息
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print('log before func : %s' % func.__name__)
            print('call func : %s' % func.__name__)
            # 装饰器自身的参数
            print('log args : %s' % p)
            print('func args : {}'.format(*args))
            result = func(*args, **kwargs)
            print('log after func : %s' % func.__name__)
            return result
        return wrapper
    return decorator

def main(p):
    print('start : %s' % main.__name__)
    print('input is : %s' % p)
    print('do something : %s' % main.__name__)
    print('end : %s' % main.__name__)

# 传递装饰器参数
decorator = log('log param')
# 初始化装饰器
log_main = decorator(main)
# 执行函数
log_main('main param')

```

### 8. 继续使用语法糖
```python
@log('log param')
def main(p):
    print('start : %s' % main.__name__)
    print('input is : %s' % p)
    print('do something : %s' % main.__name__)
    print('end : %s' % main.__name__)


main('main param')
```


### 9. 装饰器将内部的函数暴露给外部使用，是基于闭包原理
```python
def out_func():
    msg = "I'm closure"
    
    # 闭包函数
    def in_func():
        print(msg)

    return in_func


# 这里获得的就是一个闭包
closure = out_func()
# 输出 I'm closure
closure()
```
