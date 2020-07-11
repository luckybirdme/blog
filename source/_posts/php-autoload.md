---
title: PHP 自动加载
date: 2020-07-10 07:18:30
tags: PHP
---

> autoload

<!-- more -->



## 一. autoload
是php中的一个魔术方法，在代码中当调用不存在的类时会自动调用该方法。
假如现在有以下两个类文件：

```php
//Test1.php文件
<?php
class Test1{
    static function test(){
        echo "test1";
    }
}
//Test2.php文件
<?php
class Test2{
    static function test(){
        echo "test2";
    }
}

```


现在Test.php文件中要用到 Test1.php 和 Test2.php 中的类：

```php
//Test.php文件
<?php
include "Test1.php";
include "Test2.php";

Test1::test();
Test2::test();

```


用 include 或require 的问题是当我要调用的类很多的时候，include 或 require也会很多，造成代码的冗杂，而且每次执行到 Test.php 文件的时候都要加载这么多文件，有些文件还不一定用到，那就浪费了很多内存，降低效率。再者是当你某个类文件被删掉了，你还得去修改 Test.php 文件。

由于这些原因，我们用 autoload 去代替 include

```php
//Test.php文件
<?php
function __autoload($class){
    if(file_exists($class.".php")){
        require_once($class.".php");
    }else{
        die("文件不存在！");
    }
}

Test1::test();
Test2::test();
```
autoload 魔术方法的作用是当你调用不存在的类时会被自动调用，在 Test.php文件中我们调用 类Test1 和 类Test2，由于我们没有显式的引用类文件，那么系统就会自动调用 autoload 方法。


但是，到现在为止 autoload 方法基本上被废弃了！为啥呢？因为：

1、最大的缺陷就是一个文件中不允许有多个 autoload 方法，想象一下，你的项目引用了别人的一个项目，你的项目中有一个 autoload，别人的项目也有一个__autoload，这样两个 autoload 就冲突了。 解决的办法就是修改 autoload 成为一个，这无疑是非常繁琐的。
2、假如你的项目中的类根据不同的用处放在不同的文件夹中 classes 和 core，然后Test.php文件中要分别调用里面对应的类，怎么搞？这样？

```php
function __autoload($class){
    if(file_exists("classes/".$class.".php")){
        require_once("classes/".$class.".php");
    }else{
        die("文件不存在！");
    }
}
function __autoload($class){
    if(file_exists("core/".$class.".php")){
        require_once("core/".$class.".php");
    }else{
        die("文件不存在！");
    }
}

Test1::test();
Test2::test();

```
这样做的话会出现致命错误，因为 autoload 重复定义！（其实第1点的原因也是一样）


为了解决这个问题，于是就有 spl_autoload_register()

## 二. spl_autoload_register
好，到了这一步，我们先不说 spl_autoload_register，既然不用autoload，那么我们就自己定义负责类加载的函数：

```php
function my_autoload1($class){
    if(file_exists("classes/".$class.".php")){
        require_once("classes/".$class.".php");
    }else{
        die("文件不存在！");
    }
}
function my_autoload2($class){
    if(file_exists("core/".$class.".php")){
        require_once("core/".$class.".php");
    }else{
        die("文件不存在！");
    }
}

//现在我们就可以加载类文件啦
my_autoload1("Test1");
my_autoload2("Test2");

Test1::test();
Test2::test();

```

但是如上代码所示，直接调用我们的自定义类文件加载函数跟 include 有啥区别吗？这时 spl_autoload_register 就派上用场了。

显然，创造一个负责类文件加载函数不是为了让我们直接调用它，而是让PHP在需要类定义的时候为我们调用它。我们称这种功能为“自动加载”。

要开启“自动加载”功能，需要将加载函数注册到PHP中：

```php
spl_autoload_register("my_autoload1");
```


我们重新实现 Test.php:

```php
//Test.php文件
function my_autoload1($class){
    if(file_exists("classes/".$class.".php")){
        require_once("classes/".$class.".php");
    }else{
        die("文件不存在！");
    }
}
function my_autoload2($class){
    if(file_exists("core/".$class.".php")){
        require_once("core/".$class.".php");
    }else{
        die("文件不存在！");
    }
}

//将加载函数注册到PHP中
spl_autoload_register("my_autoload1");
spl_autoload_register("my_autoload2");

Test1::test();
Test2::test();
```
这样就是实现了PHP的类文件自动载入功能。




## 三. 命名空间

主要是为了避免相同的类名的冲突

```php
// 定义
// his.dog.php
namespace his\dog;
class Dog{ 
        public function __construct(){
                echo "his dog".PHP_EOL;
        }
}
// her.dog.php
namespace her\dog;
class Dog{ 
        public function __construct(){
                echo "her dog".PHP_EOL;
        }

}

// 使用
// index.php
// 加载文件
require_once('her.dog.php');
require_once('his.dog.php');
// 引入命名空间
use her\dog as her;
use his\dog as his;

$one_dog = new her\Dog();
$two_dog = new his\Dog();


```


## 四.composer
管理 PHP 类库的工具，类似 Python 的 pip，Java 的 Maven, JavaScript 的 npm

```sh
# 使用国内镜像
composer config -g repo.packagist composer https://packagist.phpcomposer.com

# 类库关系文件
composer.json
{
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
# 类库锁定文件
composer.lock

# 类库目录
vendor/your_install_lib

# 自动加载所有类库
require 'vendor/autoload.php';


# 使用类库
$log = new Monolog\Logger('name');
$handler = new Monolog\Handler\StreamHandler('app.log', Monolog\Logger::WARNING);
$log->pushHandler($handler);
$log->addWarning('Foo');

```