---
title: php 引用变量
date: 2020-04-29 07:14:09
tags: PHP
---

> & 和 @

<!-- more -->


### 变量的引用
通过 & 获取变量的引用，共用变量的内容，即共用内存

```php
function log_mem(){
    echo round(memory_get_usage()/1024/1024, 2).'MB'.PHP_EOL;
}

log_mem();

// 开辟内存保存变量
$a=array();
$aa=array();
for($i=0;$i<1000;$i++){
    $a[]=123;
    $aa[]=123;
}
log_mem();

// 变量没有改变，赋值给其他变量时，内存不变
// 普通变量赋值
$b = $a;
// 获取变量的引用
$c = &$aa;
log_mem();

// 普通变量发生变化时
// 内存发生很大变化，因为相当于重新开辟了内存
$b[]=456;
log_mem();

// 引用变量发生变化时
// 内存没有发生大变化，因为共用内存
$c[]=789;
log_mem();

```

### 函数的引用返回
和参数传递不同，函数引用返回必须在定义和调用都添加 & 符号，指出返回的是一个引用。

```php
function &load_class($class,$value)
{
        static $_classes = array();

        if (!isset($_classes[$class]))
        {
                $_classes[$class] = $value;
        }

        return $_classes;
}

// 普通函数调用
$a=load_class('one','1');
// 赋值时，只在当前有效，不会修改函数内部的变量 _classes
$a['two'] = '2';
var_dump($a);
$aa=load_class('one','1');
// 这里的返回没有 two 的赋值
var_dump($aa);

// 返回函数的引用
$b=&load_class('123','one');
// 赋值时，会同时修改函数内部的变量 _classes
$b['456']='two';
var_dump($b);
$bb=&load_class('123','one');
// 这里的返回包括 456 的赋值
var_dump($bb);

```

### PHP 函数的参数默认是传递值，如果传递引用，需要在定义函数参数时添加 & 号，使用函数时不需要添加 &

```php

$arr_one = array(
    'key'=>'one',
    'one'=>array(
        'two'=>'old'
    )
);
$arr_two = array(
    'key'=>'two',
    'one'=>array(
        'two'=>'old'
    )
);

function test_func($arr){
    $arr['one']['two']=__METHOD__;
}

function test_reference(&$arr){
    $arr['one']['two']=__METHOD__;
}

print_r($arr_one);
print_r($arr_two);

test_func($arr_one);
print_r($arr_one);

test_reference($arr_two);
print_r($arr_two);
```


### 通过 @ 屏蔽可能抛出的错误，但不推荐这样做，因为会导致无法发现致命的错误

```php
//连接数据库
function db_connect()
{
    @$db =mysql_connect('localhost','root','test');
    if(!$db){
        throw new Exception('连接数据库失败!请重试!');
    }
    mysql_select_db('book');
    return $db;
}
```