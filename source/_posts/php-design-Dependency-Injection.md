---
title: 依赖注入
date: 2020-07-10 07:58:39
tags: DesignPatterns
---

> 控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称DI）


<!-- more -->


1，依赖，直接在构造方法里实例化依赖

```php 

    class User{
        public function __construct(){
            // 依赖 database, 由内部构造方法实例化
            $this->database = new database();
        }
    }
    $user = new User();

 
```
 

2，依赖注入，在外部实例依赖之后，再注入


```php 

    class User{
        public function __construct(database $database){
            // 依赖注入 database, 由外部实例化后注入
            $this->database = $database;
        }
    }
    $database = new database(); // 依赖注入 database
    $user = new User($database);
 
```
 

3，依赖注入的好处，修改依赖时不需要修改应用类的代码


```php

	// 原始方式
    class User{
        public function __construct(){
            // 依赖 database, 由内部构造方法实例化
            $type = 'mysql'; // 修改依赖database，添加参数$type
            $this->database = new database($type);
        }
    }
    $user = new User();


   	// 依赖注入
    class User{
        public function __construct(database $database){
            // 依赖注入 database, 由外部实例化后注入
            $this->database = $database;
        }
    }
    $type = 'mysql'; // 修改依赖database，添加参数$type
    $database = new database($type); // 依赖注入 database
    $user = new User($database);

```
想像一下，如果很多应用类都依赖了database，当需要修改database时，是不是需要到每个应用类里修改这个database；
但是依赖注入只需要在应用类的外部修改database，而且修改一次就行了，不需要改动应用类的代码。



4，如果当前只有一个依赖

```php
	class User{
        public function __construct(qq $qq){
            // 依赖注入 qq, 由外部实例化后注入
            $this->qq = $qq;
        }
    }
    $qq = new qq(); // 依赖注入 qq
    $user = new User($qq);

```

 

5，新增几个依赖，传参形式变得不友好

```php
    class User{
        public function __construct(qq $qq, weibo $weibo, weixin $weixin){
            // 依赖注入 qq, weibo, weixin, 由外部实例化后注入 
            $this->qq = $qq;
            $this->weibo = $weibo;
            $this->weixin = $weixin;
        }
    }
    $qq = new qq(); // 新增依赖注入 qq
    $weibo = new weibo(); // 新增依赖注入 weibo
    $weixin = new weixin(); // 新增依赖注入 weixin
    $user = new User($qq, $weibo, $weixin);
 
```
 

6，发现新增依赖的时候，需要修改应用类的构造方法，怎么办呢？通过容器来管理依赖注入就可以解决了

```php
    class User{
        protected $container;
        public function __construct(container $container){
            // 依赖注入 container, 里面包含了所有依赖
            $this->container = $container;
        }
        public function index(){
            $qq = $this->container->get('qq');
            $weibo = $this->container->get('weibo');
            $weixin = $this->container->get('weixin');
        }
    }

    $container = new container(); // 依赖注入的容器
    $container->set('qq',function(){
        return new qq(); // 新增依赖注入 qq
    });
    $container->set('weibo',function(){
        return new weibo(); // 新增依赖注入 weibo
    });
    $container->set('weixin',function(){
        return new weixin(); // 新增依赖注入 weixin
    });
    $user = new User($container);

```
通过容器管理依赖注入，所有依赖注册到容器中，应用类只需从容器中获取依赖；
当新增依赖时，只需注册到容器中，不用改动应用类的代码


