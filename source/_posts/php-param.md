---
title: php 变量作用域
date: 2020-04-29 07:15:32
tags: PHP
---

> 变量作用域

<!-- more -->



### 变量范围
1. 局部变量，比如在函数内，在 class 里；
2. 全局变量，定义后在整个代码运行期间都有效；
3. 变量上下文范围包括 require 和 include 的文件；
4. 没有块变量的说法，即 for 循环里的变量依然是上下文有效。

- 局部变量

```php
$out_param = 'out';
function test(){
    // 局部变量
    $in_param = 'in';
    echo $in_param;
    // 无法访问函数外部的变量
    echo $out_param;
}
test();
// 无法访问函数的局部变量
echo $in_param;
echo $out_param;
```

- 全局变量通过 global 定义，或者 GLOBALS数组

```php
$a = 1;
$b = 2;
function Sum1(){
    // 不推荐这样使用
    global $a, $b;
    $b = $a + $b;
}
function Sum2()
{
    $GLOBALS['b'] = $GLOBALS['a'] + $GLOBALS['b'];
}
Sum1();
echo $b;
Sum2();
echo $b;
```

- 变量在include也生效

```php
$a = 1;
// $a 在 b.inc 里也有效
include 'b.inc';
```

### 静态变量
静态声明是在编译时解析的，静态变量仅在局部函数域中存在，但当程序执行离开此作用域时，其值并不丢失，这种特性非常重要。

- 函数内部的静态变量

```php
function count_number(){
    // 只在编译时解析，运行时不会再执行
    static $i=0;
    // 每次函数执行都会累加，因为 $i 是静态变量，其值一直保留
    $i++;
    echo $i;
    return $i;
}
count_number();
count_number();

```

- 类里面的静态变量，包括函数，可以通过 :: 直接访问，不需要 new 实例化

```php
class Person {
    static $container = array();

    static function add($name){
        if(!isset(self::$container[$name])){
            self::$container[$name]=$name;
            echo "add".PHP_EOL;
        }else{
            echo "exist".PHP_EOL;
        }
    }
}
var_dump(Person::$container);

Person::add('one');
Person::add('two');
Person::add('one');

var_dump(Person::$container);


```
