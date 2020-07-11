---
title: 单例模式
date: 2020-07-10 07:16:12
tags: DesignPatterns
---

> 单例模式

<!-- more -->


## 一，首先了解下单例模式Singleton：
1，有些应用场景，我们希望只有一个对象实例，比如全局设置变量，可以多次调用；
2，为了节省资源，我们还希望只有一个操作实例，比如连接数据库，便于减少系统开销；
于是就有了单例的需求，那么如何达到创建唯一实例了？

 
使用全局变量来实现单例是不大合理，因为可能会在程序的任何一个地方再次实例化类；
对类的实例化方法私有化或者保护，这样其他类就不能再实例它了，同时添加静态方法来实例化本身。

由此可知，单例模式Singleton的类一般具有以下特点：
1，构造方法和克隆函数声明私有化或者受保护，防止外部程序再次实例化该类
2，提供静态方法getInstance()以供外部程序获取该类的实例，同时保证实例化一次；
3，声明一个静态私有常量保存这个唯一实例

 

经典的单例对象如下:

```php
class  Singleton {
    //私有化静态常量保存唯一实例
    private static  $instance = null;
    /**
     * 私有化构造方法，保证不能被外部访问
     */
    private function __construct() {} 
    /**
     * 受保护的克隆函数，保证不能被外部访问
    */
    private  function __clone() {}
    /**
     * 提供静态方法给外部获取这个实例,并保证只有一个实例被创建
     */
    public static function getInstance() {
        if (!self::$instance) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}

$sing = Singleton::getInstance();
$sing->show_name();
$sing2 = new Singleton(); 
//Fatal error: Uncaught Error: Call to private Singleton::__construct() from invalid context in
$sing3 = clone $sing; 
//Fatal error: Uncaught Error: Call to private Singleton::__clone() from context
 
```

## 二，Java和PHP单例模式的区别
1. Java是一种编译型语言，对象可以常驻与内存中，单例会一直存在于整个应用程序的周期里，每次http请求单例都是同指向相同的内存地址；
2. PHP是一种解释型语言，每次http请求之后即PHP页面执行完毕后，所有资源都会回收，包括单例的内存地址，下回http请求会重新分配唯一的内存地址。
所以，PHP的单例模式只是针对单次页面请求，有多个应用场景需要共享同一实例资源。
