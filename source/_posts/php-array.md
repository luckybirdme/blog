---
title: php 数组函数
date: 2020-05-09 07:45:40
tags: PHP
---

> 数组

<!-- more -->



### PHP 常用数组方法

函数  |描述
-|-
分类|分割合并
array_chunk()  | 把一个数组分割为新的数组块。
array_merge()   |把一个或多个数组合并为一个数组。
array_slice()   |返回数组中被选定的部分。
array_rand()    |返回数组中一个或多个随机的键。
分类|交集差集
array_count_values()   | 用于统计数组中所有值出现的次数。
array_diff()    |比较数组，返回差集（只比较键值）。
array_diff_key()    |比较数组，返回差集（只比较键名）。
array_intersect()  | 比较数组，返回交集（只比较键值）。
array_intersect_key()  | 比较数组，返回交集（只比较键名）。
array_unique() | 删除数组中的重复值
分类|获取健值
array_column()  |返回输入数组中某个单一列的值。
array_keys()   | 返回数组中所有的键名。
array_values() | 返回数组中所有的值。
分类|回调函数
array_filter()  |用回调函数过滤数组中的元素。
array_map() |把数组中的每个值发送到用户自定义函数，返回新的值。
分类|入列出列
array_pop() |删除数组的最后一个元素（出栈）。
array_push()   | 将一个或多个元素插入数组的末尾（入栈）。
array_shift()   |删除数组中首个元素，并返回被删除元素的值。
array_unshift() |在数组开头插入一个或多个元素。
分类|数组排序
array_reverse() |以相反的顺序返回数组。
array_multisort()  | 对多个数组或多维数组进行排序。
krsort()   | 对数组按照键名逆向排序。
ksort() |对数组按照键名排序。
rsort()  |对数组排序。
sort() | 对数组排序。
分类|判断存在
array_key_exists() | 检查指定的键名是否存在于数组中。
in_array() | 检查数组中是否存在指定的值。




### 二维数组排序

```php

$a=array(
        array('num'=>2,'name'=>'jack'),
        array('num'=>3,'name'=>'luse'),
        array('num'=>1,'name'=>'pony')
);
var_dump($a);

# 获取制定 key 的 value 值
$keyValue = array_column($a, 'num');
# SORT_DESC, 倒叙 ; SORT_ASC, 顺序
array_multisort($keyValue, SORT_DESC, $a);
var_dump($a);

```


### 统计出两个数组(长度10000)中同时出现过的，并且总出现次数最多的单词

```php

$one_arr = array('one','two','three','one','one','two','one');
$two_arr = array('two','two','one','three','four','four','two');
# 计算值次数
$one_val = array_count_values($one_arr);
$two_val = array_count_values($two_arr);
print_r($one_val);
print_r($two_val);
# 求交集
$one_two_key = array_intersect_key($one_val, $two_val);
print_r($one_two_key);

$max_val = '';
$max_num = 0;
if($one_two_key){ 
    foreach ($one_two_key as $key => $value) {
        # code...
        $val_num = $one_val[$key] + $two_val[$key];
        if($val_num>$max_num){
            $max_num = $val_num;
            $max_val = $key;
        }

    }

}

echo $max_val." : ".$max_num;
```
