---
title: 工厂模式
date: 2020-07-10 07:44:38
tags: DesignPatterns
---

> 工厂模式

<!-- more -->


## 一，简单工厂模式(Simple Factory)：
模拟场景：
汽车店铺里，有些客户购买SUV，有些客户购买BWM，客户只关注现场的汽车，不会关心汽车是如何生产的，这个时候就需要一个工厂来生产SUV、BWM等之类的汽车了。

示例代码如下：

```php
//生产汽车的工具
abstract class Car{
    abstract function create();
}
// suv汽车
class Suv extends Car{
    public function create(){
        echo "create suv car"."\n";
    }
}
// bwm汽车 
class Bwm extends Car{
    public function create(){
        echo "create bwm car"."\n";
    }
}
// 生产汽车的工厂

class Car_Factoty {
    // 生产 suv 汽车
    static function create_suv() {
        return new Suv();
    }
   // 生产 bwm 汽车
    static function create_bwm() {
        return new Bwm();
    }
    
}


echo "\n-------------------------\n";
// 通过工厂生产suv汽车
$suv = Car_Factoty::create_suv();
$suv->create();

// 通过工厂生产bwm汽车
$bwm = Car_Factoty::create_bwm();
$bwm->create();

```

实际的业务场景：
在更新数据时，需要实例化一个链接数据库的类，后台数据库可能是MySQL、Oracle等，但是业务逻辑只关注更新数据，至于如何链接数据库不太关心的；这个时候就需要提供类似上述工厂的类，来创建一个链接数据库的实例，给业务层使用了。

示例代码如下：

```php
abstract class Connect_abstract{
    abstract function connect();
}
// 链接 MySQL
class MySQL extends Connect_abstract{
    public function connect(){
        echo "connect to MySQL"."\n";
    }
}
// 链接 Oracle
class Oracle extends Connect_abstract{
    public function connect(){
        echo "connect to Oracle"."\n";
    }
}

// 提供链接数据库类的工厂
class Database_Factoty {
    static function connect_mysql() {
        return new MySQL();
    }
    static function connect_oracle() {
        return new Oracle();
    }
    
}


echo "\n-------------------------\n";
// 通过工厂获得MySQL数据库的链接类
$mysql = Database_Factoty::connect_mysql();
$mysql->connect();

// 通过工厂获得Oracle数据库的链接类
$oracle = Database_Factoty::connect_oracle();
$oracle->connect();

```

由代码可看出，业务层需要链接数据库时，是通过Database_Factoty这个工厂获得链接类的；
业务层只需关注获得和使用数据库链接类，具体如何链接都是交给工厂处理。

以上的代码设计模式成为简单的工厂模式，也叫做静态的工厂方法模式（static factory method）。
下面来看看详细的工厂方法模式。

 

## 二，工厂方法模式(Factory Method):
工厂方法模式是在静态工厂方法模式基础上进行优化
直接看示例代码：

```php
abstract class Connect_abstract{
    abstract function connect();
}

class MySQL extends Connect_abstract{
    public function connect(){
        echo "connect to MySQL"."\n";
    }
}
class Oracle extends Connect_abstract{
    public function connect(){
        echo "connect to Oracle"."\n";
    }
}

// 将工厂类抽象化成接口
abstract class Database_Factoty_abstract{
    abstract function factory();
}
// 新增具体工厂类提供MySQL数据库链接类
class MySQL_Factory extends Database_Factoty_abstract{
    public function factory() {
        return new MySQL();
    }
}

// 新增具体工厂类提供Oracle数据库链接类
class Oracle_Factory extends Database_Factoty_abstract{
    public function factory() {
        return new Oracle();
    }
}

echo "\n-------------------------\n";
$mysql = new MySQL_Factory();
$mysql->factory()->connect();

$oracle = new Oracle_Factory();
$oracle->factory()->connect();

```
对比简单工厂模式，工厂方法模式把工厂类 Database_Factoty 抽象化成接口 Database_Factoty_abstract，同时新建具体的工厂类 MySQL_Factory 和 Oracle_Factory 来继承这个接口，实现所需功能。

这样的好处是：
当新增数据库类型时，只需新增一个具体的工厂类比如 MongoDB_Factory 来创建链接数据库实例，无需修改工厂类的抽象化接口，达到进一步解耦的作用。
如果是简单工厂模式，需要修改 Database_Factoty 这个工厂类，而这个类有可能被很多地方调用，关联性太强，不利于扩展和维护。

 

## 三，简单工厂模式和工厂方法模式的优点：

1，工厂模式是一种创建对象实例的模式，为了满足一些复杂或者重复的对象实例过程而设计的。
2，工厂模式也是一种解耦模式，将对象关联弱化，同时提供对象间的交流方法。
3，工厂模式的程序框架是抽象化的，具体功能有具体类实现，非常便于维护，以及进行功能扩展。