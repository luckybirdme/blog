---
title: django 介绍
date: 2020-03-22 07:54:21
tags: django
---
> django

<!-- more -->


## 一. 简介
Django 是以 Python 语言编写的 Web 应用程序框架，主要目的是简便、快速的开发 Web 网站，主要特性

- 采用 MVT 设计模式，跟 MVC 模式类似，业务逻辑分离，代码编写清晰。
- 对象关系映射 ORM ，将数据库映射到对象，操作数据库变得更简单。
- 强大的路由映射功能，支持正则匹配，以及过滤器。
- 支持多种缓存机制，包括内存，文件，数据库以及 memcached。
- 自带后台管理，几行简单的代码就可以实现管理界面。
- 完善的中英文档，详细的入门教程和示例代码。
- 清晰的错误提示，便于调试和开发。

总之，Django 是一套大而全的 Web 应用框架。

## 二. 模型 ORM
ORM 英文全称 Object Relation Mapping, 对象关系映射。面向对象编程是目前流行的编程思想，所有操作实体都是对象。关系型数据库的特点是以行和列组成表，多个表称为数据库。
通过对象也能描述和操作关系型数据库的行，列，表的数据，这种方式称为对象关系映射，映射的实体就是关系型数据库。Django ORM 如下图所示

![](/img/2020/django_orm_two.png)


### 1. 优点
- 将数据库映射成对象，只需面向对象编程，不用编写SQL。
- 将模型和数据库解耦，屏蔽了数据库差异，便于数据迁移。
- 统一添加过滤机制，比如防止SQL注入

### 2. 缺点
- 数据映射到对象过程，有性能损失
- 对于复杂的SQL，难以通过对象实现

### 3. 使用
- 创建模型
```python
# mysite/app/models.py
class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()
    def __str__(self):
        return self.name
```
- 设置模型自动加载
```python
# mysite/mysite/settings.py
INSTALLED_APPS = [
    #...
    'myapp',
    #...
]
```
- 让模型生成初始化数据的 SQL, 以及执行 SQL
```sh
# 生成SQL文件, 保存在 mysite/app/migrations 下
python manage.py makemigrations
# 执行 SQL
python manage.py migrate
```
- 使用模型操作数据库
```python
# mysite/app/views.py
from app.models import Blog
def create():
    b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
    b.save()
def select(name):
    cheese_blog  = Blog.objects.get(name=name)
# 直接执行SQL(不推荐)
def query(name):
    cheese_blog = Blog.objects.raw('SELECT * FROM blog where name="%s"' % name)
```

- 添加数据库事务操作
```python
# mysite/app/views.py
from django.db import transaction
# 针对整个方法
@transaction.atomic
def viewfunc_outsite_atomic(request):
    do_stuff()
# 在方面里面
def viewfunc_insite_atomic(request):
    do_stuff()
    with transaction.atomic():
        do_more_stuff()
```

## 三. 视图 view
Django 的视图 view 功能类似 MVC 模式中的 controller, 浏览器请求根据 url 映射到 view 具体方法，view 可以通过 ORM 模型获取数据，以及进行模板 template 渲染。

### 1. 特点
- 自由设计 url，支持正则匹配，请求分组，以及请求过滤器，比如登录校验 login_required(url)
- 常用的基础页面，比如404，也支持 json,csv,pdf 格式返回，还能通过 render 函数渲染 html
- 丰富的装饰器函数，轻松实现各种功能，比如登录校验，缓存设置，数据库事务操作

### 2. 示例
- 配置 URL 映射
```python
# mysite/mysite/urls.py
from django.urls import path
from . import views
urlpatterns = [
    path('blog/home/', views.home),
    # 传递参数
    path('articles/<int:year>/', views.year_archive),
    # 使用正则
    re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
    # 请求分组
    path('contact/', include('contact.urls')),
    # 添加过滤器
    url(r'account/info', login_required(view.account_info),),
]
```
- 编写视图方法
```python
# mysite/app/views.py
# 第一个参数默认 request
def home(request):
    pass
# 按照顺序接收参数
def year_archive(request,year):
    pass
# 使用登录校验的过滤器
@login_required
def my_func_one(request):
    pass

# 返回JSON
def my_func_json(request):
    return JsonResponse({'foo': 'bar'})
# 常用方法
def detail(request, poll_id):
    # 使用模型获取数据
    try:
        p = Poll.objects.get(pk=poll_id)
    except Poll.DoesNotExist:
    # 返回 404 页面
        raise Http404("Poll does not exist")
    # 使用 render 渲染 html
    return render(request, 'polls/detail.html', {'poll': p})
```


## 四. 模板 template
模板 template 主要是为了生成 HTML，Django 自带模板引擎 Django template language，也支持Jinja。

### 1. 优点
- 借助模板系统，生成动态的 HTML 页面
- 提供多种基础 HTML 样式，比如表单
- 自带安全机制，比如XSS，CSRF

### 2. 缺点
- 前后端代码耦合，业务逻辑复杂起来不好维护
- 目前流行更高性能，更易定制化的前端框架，比如 VueJS

### 3. 使用
- 原理
```python
from django.template import Context, Template
# 创建模板内容
t = Template('My name is {{ name }}.')
# 创建上下文变量
c = Context({'name': 'jack'})
# 进行变量替换
html = t.render(c)
# output 
# 'My name is jack.'
```
- 配置
```python
# mysite/mysite/settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

- 使用模板系统生成HTML页面
```python
def home(request, poll_id):
    # 使用 render 渲染 html
    return render(request, 'home.html',{'name': 'jack'})
```

- 模板语法
```python
# 变量
{{ my_dict.key }}
# 过滤器
{{ django|title }}

# 执行代码
{% csrf_token %}
{% if user.is_authenticated %}Hello, {{ user.username }}.{% endif %}
```

## 五. 管理台 admin
Django 自带后台管理，简单配置即可，轻松管理用户，还能直接操作DB数据。

### 1. 使用
- 配置
```python
# mysite/mysite/settings.py
INSTALLED_APPS = (
    # 自动加载 admin
    'django.contrib.admin',  
    'django.contrib.auth',    
    # ···
    'app',  #your app
)
```

- url 映射
```python
# mysite/mysite/urls.py
from django.conf.urls import  include, url
from django.contrib import admin
urlpatterns =[
    url(r'^admin/', include(admin.site.urls)), 
]
```

- 注册模型到 admin
```python
from django.contrib import admin
from .models import Question
class QuestionAdmin(admin.ModelAdmin):
    pass
admin.site.register(Question, QuestionAdmin)
```

- 创建管理员用户
```sh
# python manage.py createsuperuser
Username (leave blank to use 'root'): admin
Email address: admin@example.com
Password:
Password (again):
Superuser created successfully.
```


## 六. 缓存 cache
Django 支持多种缓存方式，缓存粒度包括全站缓存，特定视图等，也可以直接通过底层API使用缓存

### 1. 缓存方式
- 本地内存缓存
- 文件系统缓存
- 利用数据库缓存
- memcache，基于内存的缓存服务器
- 虚拟缓存，开发模式下使用，不是真的缓存

### 2. 使用
- 配置
```python
# mysite/mysite/settings.py
# memcache
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': [
            '172.19.26.240:11211',
            '172.19.26.242:11211',
        ]
    }
}

# 数据库缓存
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
    }
}

# 本地内存缓存
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
    }
}


```


- 通过中间件实现，全局缓存
```python
# mysite/mysite/settings.py
MIDDLEWARE = [
     # response 时保存缓存
    'django.middleware.cache.UpdateCacheMiddleware',
    'django.middleware.common.CommonMiddleware',
    # request 时使用缓存
    'django.middleware.cache.FetchFromCacheMiddleware',
]

```


- 缓存具体方法
```python
# mysite/app/views.py
from django.views.decorators.cache import cache_page
@cache_page(60 * 15)
def my_view(request):
    pass

# mysite/mysite/urls.py
from django.views.decorators.cache import cache_page
urlpatterns = [
    path('foo/<int:code>/', cache_page(60 * 15)(my_view)),
]
```

- 通过底层API使用缓存
```python
# mysite/app/views.py
from django.core.cache import cache
def home(request):
    p = cache.get(poll_id)
    if not p
        p = Poll.objects.get(pk=poll_id)
        cache.set(poll_id, p, 30)
```

## 七. 安全 security
### 1. XSS
Django 自带的模板系统 DTL 有过滤 XSS 的功能，对 HTML 进行转义，但是也提供 mark_safe 函数，标记安全字符串，不进行转义，这要小心使用。

### 2. CSRF
- 通过 CsrfViewMiddleware 中间件对全局请求进行 csrf_token 设置和校验
- 对某些方法单独使用装饰器 csrf_protect 校验，或者忽略 csrf_exempt
- 模板在 form 表单添加 csrf_token ，AJAX 请求则需要 JS 设置 csrftoken

### 3. SQL 注入
Django ORM 对 SQL 注入有防范机制，除非你使用 raw 函数直接执行 SQL

### 4. 页面劫持 ClickJacking
Django 通过中间件 XFrameOptionsMiddleware 设置 HTTP 请求头 X-Frame-Options，防止页面被恶意网站页面通过 iframe 嵌套

## 八. 国际化和本地化
国际化和本地化的目标是让同一站点为不同的用户提供定制化的语言和格式服务，Django 借助 gettext() 函数实现以上功能。

### 1. 使用
- 配置
```python
# mysite/mysite/settings.py
LANGUAGES = (
    ('en', ('English')),
    ('zh-hans', ('中文简体')),
)
```

- 通过 gettext函数 标记需要翻译的文本
```python
# mysite/mysite.views.py
from django.http import HttpResponse
from django.utils.translation import gettext as _
def my_view(request):
    output = _("Welcome to my site.")
    return HttpResponse(output)
    
```

- 在 template 里标记需要翻译的文本
```python
# template/home.html
<title>{% trans "This is the title." %}</title>

```


- 遍历 Django 的应用目录，搜集带翻译的文本，并在目录 locale/{LANG}/LC_MESSAGES/ 生成文件 django.po，${LANG} 对应语言
```python
# 生成中文简体
django-admin makemessages -l zh_hans
```

- 打开文件 locale/zh_hans/LC_MESSAGES/django.po, 翻译每个 msgstr
```pythno
#: path/to/python/module.py:23
msgid "Welcome to my site."
msgstr "欢迎来到我的网站"
```


- 最后将翻译文件编译成二进制文件，网站即可根据不同用户选择，自动切换不同语言
```python
django-admin compilemessages
```

## 九. 单元测试 test
Django 测试组件继承于 Python 的 unittest 类，使用方法也大体相似。


### 1. 使用
- 编写测试文件
```python
# mysite/app/tests.py
from django.test import TestCase
from app.models import Blog
class BlogTestCase(TestCase):
    # 测试前执行初始化
    def setUp(self):
        Blog.objects.create(name="hello", content="world")
    # test开头的函数都将会被调用
    def test_blog_create(self):
        b = Blog.objects.get(name="hello")
        # 利用断言测试
        self.assertEqual(b.content, 'world')
    # 测试后清理数据
    def tearDown(self):
        Blog.objects.filter(name='hello').delete()
```

- 运行测试文件
```sh
# Run all the tests found within the 'app' package
$ ./manage.py test app

# Run all the tests in the app.tests module
$ ./manage.py test app.tests

# Run just one test case
$ ./manage.py test app.tests.BlogTestCase

# Run just one test method
$ ./manage.py test app.tests.BlogTestCase.test_blog_create

```




