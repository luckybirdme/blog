---
title: PHP 类特性
date: 2020-04-28 07:16:36
tags: PHP 
---

> class

<!-- more -->


### 类是模板，对象是根据模板生成的，这个过程叫做实例化

```php

class TestClass{
    public $name='';
    public function __construct($name){
        echo "__contruct".PHP_EOL;
        $this->name=$name;
    }
    public function show_name(){
        echo $this->name;
    }
}
// 将类实例化成对象
$test_class = new TestClass('test');
// 访问方法
$test_class->show_name();

```

### 魔术方法

```php

class TestClass{
    public $name='';
    // 使用尚未被定义的类时自动调用,脚本引擎在 PHP 出错失败前有了最后一个机会加载所需的类
    public function __autoload(){
        echo __METHOD__.PHP_EOL;
    }
    // 当一个类实例化对象时调用
    public function __construct($name){
        echo __METHOD__.PHP_EOL;
        $this->name=$name;
    }
    // 对象销毁时调用
    public function __destruct(){
        echo __METHOD__.PHP_EOL;
    }
    // 获取不存在属性时调用
    public function __get($key){
        echo __METHOD__.PHP_EOL;
        echo $key.PHP_EOL;
    }
    // 获取不存在方法时调用
    public function __call($func,$argv){
        echo __METHOD__.PHP_EOL;
        echo $func.PHP_EOL;
    }
    // 克隆对象时调用
    public function __clone(){
        echo __METHOD__.PHP_EOL;
        $this->name="clone";
    }
    // 普通方法
    public function show_name(){
        echo __METHOD__.PHP_EOL;
        echo $this->name.PHP_EOL;
    }
}

$test_class = new TestClass('test');
$test_class->show_name();
$test_class->name_not_exist;
$test_class->func_not_exist();

$clone_class = clone $test_class;
$clone_class->show_name();

$test_class->show_name();

```


### 类的继承，只能继承一个父类

```php

class TestClass{
    public $name='name';

    public function show_name(){
        echo __METHOD__.PHP_EOL;
    }
}
// 通过 extends 继承父类，只能继承一个
class TestExtend extends TestClass{
    public function __construct(){
        echo __METHOD__.PHP_EOL;
    }

}

$test_extend = new TestExtend();
$test_extend->show_name();

```


### 变量分类
- private, 私有变量，只允许自己调用
- protected, 保护变量，允许自己和子类调用
- public, 共有变量，谁都能调用

```php

class TestClass{
    protected $name='name';
    private $age = 10;
    public function show_name(){
        echo __METHOD__.PHP_EOL;
    }
    public function show_age(){
        echo __METHOD__.PHP_EOL;
        echo $this->age.PHP_EOL;
    }
}

class TestExtend extends TestClass{
    public function __construct(){
        echo __METHOD__.PHP_EOL;
    }

    public function get_name(){
        echo __METHOD__.PHP_EOL;
        echo $this->name.PHP_EOL;
    }
    public function get_age(){
        echo __METHOD__.PHP_EOL;
        echo $this->age.PHP_EOL;
    }

}

$test_class =  new TestClass();
// 只能自己访问私有属性
$test_class->show_age();

$test_extend = new TestExtend();
$test_extend->show_name();
// 访问父类的保护属性
$test_extend->get_name();
// 无法访问父类的私有属性
$test_extend->get_age();

```

- static, 静态变量，类不需要实例化对象，通过冒号(::)即可访问

```php
class TestClass{
    public static $school="static";
    public static function show_name(){
        echo __METHOD__.PHP_EOL;
    }
}
// 静态方法
TestClass::show_name();
// 静态属性
echo TestClass::$school.PHP_EOL;

```


- final, 最终变量，不允许被子类被继承，不允许被子类重写方法

```php
<? 
//声明一个final类Math 
final class Math{ 
    public static $pi = 3.14; 
    public function __toString()
    { 
        return "这是Math类。"; 
    } 
　　public final function max($a,$b)
　　{ 
    　　return $a > $b ? $a : $b ; 
　　} 
} 
//声明类SuperMath 继承自 Math类 
class SuperMath extends Math { 
} 
//执行会出错，final类不能被继承。 

//声明类SuperMath 继承自 Math类 
class SuperMath extends Math{ 
    public final function max($a,$b){} 
} 
//执行会出错，final方法不能被重写。 

?>
```


### 单实例模式：防止重复实例化，避免大量的new操作，减少消耗系统和内存的资源，使得有且仅有一个实例对象

```php

# 单实例的类
class Singleton
{
    //创建静态私有的变量保存该类对象
    private static $instance;

    //防止使用new直接创建对象
    private function __construct(){}

    //防止使用clone克隆对象
    private function __clone(){}

    public static function getInstance()
    {
        //判断$instance是否是Singleton的对象，不是则创建
        if (!self::$instance instanceof self) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    public function show_name()
    {
        echo "Singleton";
    }
}

$sing = Singleton::getInstance();
$sing->show_name();
$sing2 = new Singleton(); 
//Fatal error: Uncaught Error: Call to private Singleton::__construct() from invalid context in
$sing3 = clone $sing; 
//Fatal error: Uncaught Error: Call to private Singleton::__clone() from context

```


### 工厂模式：用工厂方法代替new操作的一种模式，如果需要更改所实例化的类名，只需在工厂方法内修改，不需逐一寻找代码中具体实例化的地方

```php
/**
 * 测试类一
  */
class demo1
{
    //定义一个test1方法
    public function test1()
    {
        echo '这是demo1类的test1方法'.PHP_EOL;
    }
}
/**
 * 测试类二
  */
class demo2
{
    //定义一个test2方法
    public function test2()
    {
        echo '这是demo2类的test2方法'.PHP_EOL;
    }
}
/**
 * 工厂类
 */
class Factoty
{
    // 根据传参类名，创建对应的对象
    static function createObject($className)
    {
        return new $className();
    }
}
/**
 * 通过传类名，调用工厂类里面的创建对象方法
 */
$demo = Factoty::createObject('demo1');
$demo->test1();             //输出这是demo1类的test1方法
$demo = Factoty::createObject('demo2');
$demo->test2();            //输出这是demo2类的test2方法

```



### 命名空间，为了避免相同的类名的冲突

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


### composer, 管理 PHP 类库的工具，类似 Python 的 pip，Java 的 Maven, JavaScript 的 npm

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