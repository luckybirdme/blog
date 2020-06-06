---
title: python uWSGI
date: 2020-03-19 07:54:21
tags: Python
---
> WSGI, uWSGI, uwsgi

<!-- more -->

## 一. WSGI, Python Web Server Gateway Interface
Python web 服务器网关接口，定义Web服务器(nginx)与Python应用程序之间交互的协议。

![](/img/2020/WSGI.png)


下面简单实现 WSGI 协议

### 1. 应用程序，实现 application 函数，也可以使用实现 \_\_call\_\_ 方法的对象
```python
# wsgi_app.py
# env 是请求的参数
# response 是用于返回的对象
def application(env, response):
    # 直接返回 200 成功
    response('200 OK',[('Content-Type','text/html')])
    body=''
    # 打印请求参数
    for k,v in env.items():
        body+='%s %s\n' % (k,v)
    return [body]
```

### 2. 服务器，直接使用 Python 内置模块 wsgiref，并将 application 作为回调函数
```python
# wsgi_server.py
from wsgiref.simple_server import make_server
from wsgi_app import application

httpd = make_server('0.0.0.0',8000,application)
print('Serving HTTP on port 8000...')
httpd.serve_forever()
```

### 3. 测试服务器和应用程序
```sh
# python wsgi_server.py
# curl http://localhost:8000

SERVER_SOFTWARE WSGIServer/0.1 Python/2.7.5
REQUEST_METHOD GET
SERVER_PROTOCOL HTTP/1.1
LANG zh_CN.UTF-8
SERVER_PORT 8000
```



## 二. uWSGI
实现 WSGI 协议的 Web 服务器，同时支持HTTP协议，FastCGI协议，SCGI协议，以及独有的uwsgi协议，它是一种二进制协议,利用TCP通信，占用内存少，被认为是上述协议中性能最好的，就是命名有点奇怪。
uWSGI 不仅支持 Python 应用程序，还支持 PHP,Lua,Go 等语言，它本质只是定义服务器和应用程序之间的交互协议，是用来管理应用程序的平台组件。

![](/img/2020/uWSGI.png)


### 1. 为什么使用 uWSGI 管理 Python 应用程序？
- 实现了 WSGI 协议，也是 uWSGI 实现的第一个可用插件协议。
- uwsgi 协议性能更高，python内置的wsgiref只能用来开发测试。
- 方便管理 Python 应用程序，而且提供了监控，集群，负载均衡等功能。

### 2. 为什么使用 NGINX 作为 uWSGI 的前置组件？
- 诚然 uWSGI 实现了 HTTP 协议，但 NGINX 是专注于 HTTP 转发的高性能 Web 服务器，尤其是负载均衡，缓存管理，以及静态资源处理方面。
- NGINX 在安全漏洞做得更完善，黑白名单更容易配置，暴露在公网时需要考虑更多的安全机制，而uWSGI 还在努力完善中。


## 三. NGINX + uWSGI + Django
Django 是以 Python 语言编写的 Web 应用程序框架，采用 MVT 设计模式，M是模型，V是控制器，T是模板，跟 MVC 模式类似。它同时实现了 WSGI 协议，提供了 返回 application 的文件 wsgi.py。




### 1. 启动 Django 内置的 wsgiref 服务器
```sh
# 本地调试使用
$ python manage.py runserver 0.0.0.0:8000
```

### 2. 使用 uWSGI 服务器来管理 Django ，采用 http 协议
```sh
# 安装 uWSGI 服务器
$ pip install uwsgi
# 使用 uWSGI
$ uwsgi --http 127.0.0.1:8000 --chdir /www/django --wsgi-file /www/django/mysite/wsgi.py
# 测试
$ curl -i http://127.0.0.1:8000
```

### 4. 使用 uWSGI 更高性能的本地协议 uwsgi 来启动
```sh
$ uwsgi --socket 127.0.0.1:8000 --chdir /www/django --wsgi-file /www/django/mysite/wsgi.py
```

### 5. 添加 NGINX 作为 uWSGI 的前置组件，处理 HTTP 请求
```sh
location / {
    include uwsgi_params;
    # 指向 uWSGI
    uwsgi_pass 127.0.0.1:8000;
}
```

[参考uWSGI官网](https://uwsgi-docs.readthedocs.io/en/latest/)




