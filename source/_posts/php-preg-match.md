---
title: PHP 正则匹配
date: 2020-07-09 22:34:27
tags: PHP
---

> preg_match

<!-- more -->


1. preg_match 
- 只匹配一次
- $matches 返回是一维数组，第一个参数是匹配到完整字符串；第二参数起，是通过(.\*)获取到的字符串变量

```php
// preg_match ( string $pattern , string $subject [, array &$matches [, int $flags = 0 [, int $offset = 0 ]]] ) : int

$str='<ul class="attr">
    <li>
        <a href="www.baidu.com">百度baidu</a>
        <a href="www.tecent.com">腾讯tengxun</a>
        <a href="www.alibaba.com">阿里巴巴alibaba</a>
    </li>
</ul>';

preg_match('/href="(.*)">(.*)</',$str,$arr);
print_r($arr);

/*
Array
(
    [0] => href="www.baidu.com">百度baidu<
    [1] => www.baidu.com
    [2] => 百度baidu
)
*/
```

2. preg_match_all
- 匹配所有符合条件的字符串
- $matches 返回是二维数组，第一个参数是匹配到完整字符串，可能存在多个，所以是个数组；第二参数起，是通过(.\*)获取到的字符串变量，也是数组形式

```php
// preg_match_all ( string $pattern , string $subject [, array &$matches [, int $flags = PREG_PATTERN_ORDER [, int $offset = 0 ]]] ) : int

preg_match_all('/href="(.*)">(.*)</',$str,$arr);
print_r($arr);
/*
Array
(
    [0] => Array
        (
            [0] => href="www.baidu.com">百度baidu<
            [1] => href="www.tecent.com">腾讯tengxun<
            [2] => href="www.alibaba.com">阿里巴巴alibaba<
        )

    [1] => Array
        (
            [0] => www.baidu.com
            [1] => www.tecent.com
            [2] => www.alibaba.com
        )

    [2] => Array
        (
            [0] => 百度baidu
            [1] => 腾讯tengxun
            [2] => 阿里巴巴alibaba
        )

)
*/

```





