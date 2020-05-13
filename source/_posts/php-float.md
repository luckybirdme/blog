---
title: php 浮点数
date: 2020-05-09 07:51:21
tags: PHP
---

> float

<!-- more -->


### 浮点数问题
- 浮点数9.45，底层实际上是 9.44999..., 9.45*100的浮点数计算后是 944.999...
- 永远不要相信浮点数结果精确到了最后一位，也永远不要比较两个浮点数是否相等。如果确实需要更高的精度，应该使用任意精度数学函数或者 gmp 函数。

```php
$a = 9.42*100;
var_dump($a); 
var_dump(intval($a));
// double(942)
// int(942)

$b = 9.45*100;
var_dump($b); 
var_dump(intval($b));
//double(945)
//int(944)

$c = 0.35*100;
var_dump($c); 
var_dump(intval($c));
//double(35)
//int(35)

$d = 0.58*100;
var_dump($d); 
var_dump(intval($d));
//double(58)
//int(57)

```