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


### 子类的权限一定要大于等于父类的权限，比如父类中成员方法为public时，子类继承过来的成员方法只能为public；而父类中成员方法为protected时，子类中继承的权限只能为protected或者public


```php
class person{
    protected $name;
    protected $age;
    public $sex;
    function __construct($name,$age,$sex){
        $this->name=$name;
        $this->age=$age;
        $this->sex=$sex;
    }
    //父类中say()方法的权限为protected
    protected function say(){
        echo "我的名字是{$this->name},我的年纪是：{$this->age},我的性别为：{$this->sex}<br>";
    }
}
class student extends person{
    public $school;
    function __construct($name,$age,$sex,$school){
        parent::__construct($name,$age,$sex);
        $this->school=$school;
    }
    //父类为protected,子类中继承的方法的权限则只能为public或者protected
    protected function say(){
        parent::say();
        echo "我所在的学校是{$this->school}";
    }
    function study(){
        $this->say();
        echo "111111111";
    }
}
$stu=new student("刘仁","20","男","大梁工业大学");
$stu->study();

```

### 对象的多态性是指在父类中定义的属性或行为被子类继承之后，可以具有不同的数据类型或表现出不同的行为。

```php

class animal{
    function can(){
        echo "this function weill be re-write in the children";
    }
}
class cat extends animal{
    function can(){
        echo "I can climb";
    }
}
class dog extends animal{
    function can(){
        echo "I can swim";
    }
}
function test($obj){
    $obj->can();
}
test(new cat());
test(new dog());
?>
```