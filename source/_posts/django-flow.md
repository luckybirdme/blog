---
title: django 核心流程解读
date: 2020-03-22 07:54:21
tags: django
---
> django

<!-- more -->

## 一. Django 请求执行过程
### 1. 请求执行主流程图
![](/img/2020/django_main.png)


### 2. 中间件执行流程图
![](/img/2020/django_mw.png)


## 二. Django 代码执行解读

### 如果采用 uWSGI 服务器，那么 Django 的入口文件就是 wsgi.py。

```python
# mysite/mysite/wsgi.py
import os
from django.core.wsgi import get_wsgi_application
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings')
# 获取网站的 application
application = get_wsgi_application()
```

### wsgi.py 首先会通过 setup 函数执行初始化，最后返回一个 WSGIHandler。

```python
# django/core/wsgi.py
def get_wsgi_application():
    # 初始化网站，加载 setting, model
    django.setup(set_prefix=False)
    return WSGIHandler()
```



###  setup 函数初始化主要加载 INSTALLED_APPS，比如 app 的 model 

```python
# django/__init__.py
def setup(set_prefix=True):
    # 加载  INSTALLED_APPS
    apps.populate(settings.INSTALLED_APPS)

# django/apps/registry.py
class Apps:
    def populate(self, installed_apps=None):
        with self._lock:
            # 加载 app 
            # Phase 1: initialize app configs and import app modules.
            for entry in installed_apps:
                if isinstance(entry, AppConfig):
                    app_config = entry
                else:
                    app_config = AppConfig.create(entry)

            # 加载模型
            # Phase 2: import models modules.
            for app_config in self.app_configs.values():
                app_config.import_models()
```


### WSGIHandler 实现了 WSGI 协议，这是 Django 执行流程的核心函数，首先会串联中间件，接着初始化请求，然后执行请求，获得返回结果。
```python
# django/core/handlers/wsgi.py
# 实现 WSGI 协议
class WSGIHandler(base.BaseHandler):
    def __init__(self, *args, **kwargs):
        # 初始化所有中间件
        self.load_middleware()
    def __call__(self, environ, start_response):
        # 请求的参数
        request = self.request_class(environ)
        # 执行请求的方法
        response = self.get_response(request)
        # 返回结果
        start_response(status, response_headers)
        return response
```

### 加载中间件时，会把所有中间件初始化成嵌套的链式，好像穿过洋葱一样，从外层执行到里层，再从里层执行到外层。而请求 url 的具体方法 function，就在洋葱的最里层。
```python
# django/core/handlers/base.py
def load_middleware(self):
    # _get_response 函数就是执行请求 url 的具体方法 view_function。
    # 把这个方法，放在链式最里层，当做中间件来处理
    handler = convert_exception_to_response(self._get_response)
    # reversed 把中间件倒叙排列，然后一个个地加到链式上
    for middleware_path in reversed(settings.MIDDLEWARE):
        middleware = import_string(middleware_path)
        try:
            # 将下一个中间件作为参数传递给当前中间件
            # 相当于中间件初始化函数 __init__(self,get_response) 的 get_response 参数
            # 这是中间件嵌套调用的关键所在
            mw_instance = middleware(handler)
        except MiddlewareNotUsed as exc:
            continue
        
        # 中间件的钩子函数 process_view
        # insert 插入到前面，所以会按照顺序排列。
        if hasattr(mw_instance, 'process_view'):
            self._view_middleware.insert(0, mw_instance.process_view)

        # 中间件的钩子函数 process_template_response
        # append 放到后面，所以会按照倒序排列。
        if hasattr(mw_instance, 'process_template_response'):
            self._template_response_middleware.append(mw_instance.process_template_response)
        # 中间件的钩子函数 process_exception
        # append 放到后面，所以会按照倒序排列。
        if hasattr(mw_instance, 'process_exception'):
            self._exception_middleware.append(mw_instance.process_exception)

        # 添加到中间件到链式上，最终排列格式如下：
        # mw_instance 3 -> function
        # mw_instance 2 -> mw_instance 3 -> function
        # mw_instance 1-> mw_instance 2 -> mw_instance 3 -> view_function
        handler = convert_exception_to_response(mw_instance)
    # 返回链式
    self._middleware_chain = handler
```

### convert_exception_to_response 函数通过参数 get_response 嵌套调用，把所有中间件 mw_instance 串联起来。如果有个中间件执行异常，会立即暂停嵌套调用，后续的中间件也就不会再执行了。
```python
# django/core/handlers/exception.py
def convert_exception_to_response(get_response):
    @wraps(get_response)
    # 定义闭包，以便中间件后续调用
    def inner(request):
        try:
            # get_response 是中间件初始化成链式时传递的参数 mw_instance
            # 因此 get_response() 相当于执行中间件
            # 链式上的中间件就是通过此方法嵌套调用，直到执行完成所有中间件
            response = get_response(request)
        except Exception as exc:
            # 不再执行中间件 get_response
            # 暂停嵌套调用，后续中间件不会执行
            response = response_for_exception(request, exc)
        return response
    return inner
```


### 中间件链式由 WSGIHandler 函数调用 get_response 函数 开始触发，执行第一个中间件，中间件自己进行后续的嵌套调用。
```python
# django/core/handlers/base.py
def get_response(self, request):
    # 开始调用中间件链式
    response = self._middleware_chain(request)
    return response
```


### 每个中间件初始化时，会接收下一个中间件对象作为参数，以便中间件自己执行完后，会接着调用下一个中间件， 直到执行完链式上的所有中间件为止。
```python
# mysite/app/middleware/one_middleware.py
# 定义中间件的例子
class OneMiddleware:
    # 中间件通过 load_middleware 时，执行了 mw_instance = middleware(handler)
    # 上面的 handler 就是这里的 get_response 参数
    def __init__(self, get_response):
        # 中间件初始化，接收下一个中间件对象
        self.get_response = get_response
    def __call__(self, request):
        # 在执行请求 url 的具体方法前，执行自身中间件的逻辑
        do_something_before_request()

        # 执行下一个中间件，实现中间件的嵌套调用，并且透传请求参数 request
        response = self.get_response(request)
        return response
```


### 中间件链式由外层执行到最里层后，最终会执行请求 url 的具体 view_function ，即 \_get_response 方法，首先会进行 url 路由匹配，接着执行中间件的钩子函数 process_view，如果任意一个钩子函数有 response，则会停止执行后续的代码，立即返回给前端。
```python
# django/core/handlers/base.py
def _get_response(self, request):
    # 进行路由匹配
    if hasattr(request, 'urlconf'):
        urlconf = request.urlconf
        set_urlconf(urlconf)
        resolver = get_resolver(urlconf)
    else:
        resolver = get_resolver()
    resolver_match = resolver.resolve(request.path_info)
    # 获取具体方法 callback, 参数 callback_args, callback_kwargs 
    callback, callback_args, callback_kwargs = resolver_match
    request.resolver_match = resolver_match

    # 按顺序，执行所有中间件的 process_view 函数
    # _view_middleware 变量在中间件初始化函数 load_middleware 时赋值了
    for middleware_method in self._view_middleware:
        response = middleware_method(request, callback, callback_args, callback_kwargs)
        if response:
            break
```

### 终于到了关键的步骤，开始执行请求 url 的具体 view_function ；如果是渲染视图，将会执行中间件的钩子函数 process_template_response；如果执行请求 url 的具体 view_function 或者 render 渲染 HTML 异常，会触发中间件的钩子函数 process_exception，这些钩子函数跟 process_view 类似，一有 response 则停止执行后续代码，立即返回前端。

```python
# django/core/handlers/base.py
def _get_response(self, request):

    if response is None:
        # 数据库采取事物操作，还是自动提交
        wrapped_callback = self.make_view_atomic(callback)
        try:
            # 执行请求 url 的具体方法 view_function，即 callback
            response = wrapped_callback(request, *callback_args, **callback_kwargs)
        except Exception as e:
            # 执行所有中间件的钩子函数 process_exception
            response = self.process_exception_by_middleware(e, request)

    # Complain if the view returned None (a common error).
    if response is None:
        pass
    # 如果是渲染视图
    elif hasattr(response, 'render') and callable(response.render):
        # 倒叙方式
        # 执行所有中间件的钩子函数 process_template_response
        # _template_response_middleware 变量在中间件初始化函数 load_middleware 时赋值了
        for middleware_method in self._template_response_middleware:
            response = middleware_method(request, response)
        try:
            response = response.render()
        except Exception as e:
            # 执行所有中间件的钩子函数 process_exception
            response = self.process_exception_by_middleware(e, request)
    logger.debug('response')
    return response

def process_exception_by_middleware(self, exception, request):
    # 倒序方式
    # _exception_middleware 变量在中间件初始化函数 load_middleware 时赋值了
    for middleware_method in self._exception_middleware:
        response = middleware_method(request, exception)
        if response:
            return response
    raise
```



### 请求 url 的具体方法通常是 view 里的方法，view 可以通过 model 操作 DB，也可以利用 template 渲染 HTML 页面
```python
# mysite/app/views.py
def detail(request, question_id):
    try:
        # 通过 Question 模型对象获取 DB 数据
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    # 利用 render 渲染 HTML 页面
    return render(request, 'polls/detail.html', {'question': question})
```


### 当执行完请求 url 的具体方法获得返回结果 response，刚好从中间件链式的最外层执行到了最里层，接着又从最里层向最外层执行每个中间件。
```python
# mysite/app/middleware/one_middleware.py
class OneMiddleware:
    # 中间件通过 load_middleware 时，执行了 mw_instance = middleware(handler)
    # 上面的 handler 就是这里的 get_response 参数
    def __init__(self, get_response):
        # 中间件初始化，接收下一个中间件对象
        self.get_response = get_response
    def __call__(self, request):
        # 在执行请求 url 的具体方法前，执行自身中间件的逻辑
        # 从中间件链式的最外层，执行到最里层
        do_something_before_request()

        # 执行下一个中间件，并且透传请求参数 request
        # 最后一个中间件是请求 url 的具体方法。
        response = self.get_response(request)


        # 从中间件链式的最里层，向最外层执行。
        do_something_after_response()

        return response
```

### 回到 Django 执行流程的核心函数 WSGIHandler，它会触发第一个中间件执行，中间件链式由外到里，再由里到外，嵌套执行，最后返回结果给 WSGIHandler，WSGIHandler 再给 uWSGI 服务器，到此执行完一个请求的全部流程。
```python
# 实现 WSGI 协议
# django/core/handlers/wsgi.py
class WSGIHandler(base.BaseHandler):
    def __call__(self, environ, start_response):
        # 请求的参数
        request = self.request_class(environ)
        # 执行所有中间件和请求的方法，由外到里再由里到外，请求的方法在最里层。
        response = self.get_response(request)
        # 返回结果
        start_response(status, response_headers)
        return response
```










