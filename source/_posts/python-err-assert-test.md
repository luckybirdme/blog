---
title: python 错误 断点 测试
date: 2020-02-04 21:37:41
tags: Python
---

> try execpt else finally
> assert

<!-- more -->


### 1. 错误捕捉

```python
print('start')
try:
    print('try')
    r = 10 / 0
    print('result:', r)
except Exception as e:
    print('except:', e)
finally:
    print('finally')
print('end')

```

### 2. 捕捉具体报错

```python
print('start')
try:
    print('try')
    # r = 10 / int('2')
    # r = 10 / int('a')
    r = 10 / int('0')
    print('result:', r)
# 捕捉所有错误
# except Exception as e:
#    print('except:', e)
# 捕捉指定错误
except ValueError as e:
    print('ValueError:', e)
except ZeroDivisionError as e:
    print('ZeroDivisionError:', e)
# 没有报错时会执行
else:
    print('no error')
# 不管有没报错，最后都会执行
finally:
    print('finally')
print('end')
```


### 3. 断点，返回 false 会抛出异常

```python
def foo(s):
    print('start')
    n = int(s)
    # 断言，false 会中断
    assert n != 0, 'n is zero!'
    print('end')
    return 10 / n

foo('1')
foo('0')
```

### 4. 断点测试
```python
import re
def is_valid_email(addr):
    # 测试不通过
    # return re.match(r'\w*@\w*\.\w*',addr)
    return re.match(r'[\w|\.]*@\w*\.\w*',addr)

# 测试:
assert is_valid_email('someone@gmail.com')
assert is_valid_email('bill.gates@microsoft.com')
assert not is_valid_email('bob#example.com')
assert not is_valid_email('mr-bob@example.com')
print('ok')

```

### 5. 单元测试

```python

import unittest

# 具体功能
class Dict(dict):

    def __init__(self, **kw):
        super().__init__(**kw)

    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Dict' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value

# 单元测试
class TestDict(unittest.TestCase):

    def test_init(self):
        d = Dict(a=1, b='test')
        # 借助断言测试
        self.assertEqual(d.a, 1)
        self.assertEqual(d.b, 'test')
        self.assertTrue(isinstance(d, dict))

    def test_key(self):
        d = Dict()
        d['key'] = 'value'
        self.assertEqual(d.key, 'value')

    def test_attr(self):
        d = Dict()
        d.key = 'value'
        self.assertTrue('key' in d)
        self.assertEqual(d['key'], 'value')

    def test_keyerror(self):
        d = Dict()
        # 捕捉特殊报错
        with self.assertRaises(KeyError):
            value = d['empty']

    def test_attrerror(self):
        d = Dict()
        with self.assertRaises(AttributeError):
            value = d.empty
if __name__ == '__main__':
    unittest.main()
    
```