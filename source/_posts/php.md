---
title: PHP 知识点
date: 2020-03-28 21:06:03
tags: PHP
---

> php


<!-- more -->


### 大小写敏感
所有用户定义的函数、类和关键词（例如 if、else、echo 等等）都对大小写不敏感，除非人为检查限制。
但是所有定义的变量，对大小写敏感，包括 defined 的常量名，以及数组 key。

```php
IF(true){
	ECHO 'YES';
}ELSE{
	ECHO 'NO';
}
$one='one';
$ONE='ONE';
var_dump($one);
var_dump($ONE);
```

### 判断是否存在字符

```php
$a = 'How are you?';
if (strpos($a, 'are') !== false) {
    echo 'true';
}
```


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


### require 和 include 区别
- require 找不到文件或者返回严重错误之后脚本就会终止执行
- include 无法找到文件时，PHP 会继续执行下去


### 文件读写支持多种操作模式

模式|描述
--|--
r|	打开文件为只读。文件指针在文件的开头开始。
w|	打开文件为只写。删除文件的内容或创建一个新的文件，如果它不存在。文件指针在文件的开头开始。
a|	打开文件为只写。文件中的现有数据会被保留。文件指针在文件结尾开始。创建新的文件，如果文件不存在。
x|	创建新文件为只写。返回 FALSE 和错误，如果文件已存在。
r+|	打开文件为读/写、文件指针在文件开头开始。
w+|	打开文件为读/写。删除文件内容或创建新文件，如果它不存在。文件指针在文件开头开始。
a+|	打开文件为读/写。文件中已有的数据会被保留。文件指针在文件结尾开始。创建新文件，如果它不存在。
x+|	创建新文件为读/写。返回 FALSE 和错误，如果文件已存在。

```php
$myfile = fopen("test.txt", "w") or die("Unable to open file!");
fwrite($myfile,'Hello');
fclose($myfile);
$myfile = fopen("test.txt", "r") or die("Unable to open file!");
echo fread($myfile,filesize("test.txt"));
fclose($myfile);

```


### cookie
cookie 保存在浏览器，用于记录用户的基本信息，在每次发起请求到服务端时，都会携带上 cookie，服务端根据 cookie 识别是哪个用户的请求。

### session 
session 是保存在服务端的，用于记录请求用户的基本信息，服务端根据 cookie 传递的 session_id，就能拿到具体的 session 信息。服务端保存 session 的目录在 php.ini 中 session.save_path 设置，内容格式为:"变量名\|类型:长度:值;"

```php
# 服务端设置 cookie，将会自动传给浏览器 
setcookie("session_id",md5(time()),time()+2*7*24*3600);
setcookie("user_name",'jack',time()+2*7*24*3600);

# 删除 cookie
setcookie("user_name",'',time()-1);

# 服务端设置 session
# 开启Session
session_start();
# 设置 session
$_SESSION['user_name'] = 'jack';

# 删除某个 session 值
unset($_SESSION['user_name']);

# 清空所有 session
session_destroy();
```


### 捕捉异常

```php

// 执行代码
try{
    echo 'ok';
    //throw new Exception("Error");
}
//捕获异常
catch(Exception $e){
    echo 'Message: ' .$e->getMessage();
}
```


### 方便的过滤器

```php
# 判断 $_GET 是否存在 email 变量
if(!filter_has_var(INPUT_GET, "email")){
    echo("Input type does not exist");
}else{
	# 判断 $_GET 的 email 变量是否符合格式
    if (!filter_input(INPUT_GET, "email", FILTER_VALIDATE_EMAIL)){
        echo "E-Mail is not valid";
    }else{
        echo "E-Mail is valid";
    }
}
```



### 客户端IP地址和服务器端IP地址。

```php
# 客户端
$REMOTE_ADDR = $_SERVER['REMOTE_ADDR']
# 服务端
$SERVER_ADDR = $_SERVER['SERVER_ADDR']
```

### 序列化和反序列化作用
- 将数组保存到数据库，要用时再拿出来

```php
$a = array(
    'key'=>'val'
    'more'=>array(
        'key'=>'val'
    )
);

# 序列化成字符串
$s = serialize($a);
# 反序列化
$aa = unserialize($s);

```

- 前后端交互通过 JSON 格式传递数据

```php
$o = array(
    'state'=>0,
    'data'=>array(
        'key'=>'val'
    )
);

# ajax 请求返回 json 结果
echo json_encode($o);

```


### PHP 编码转换
1. iconv ( string $in_charset , string $out_charset , string $str )
- 需要指定输入和输出的编码
- 为了生僻字转换异常，需要添加 IGNORE 

``` php
$outstr = iconv("UTF-8","GB2312//IGNORE",$data);
```

2. mb_convert_encoding ( string $str , string $to_encoding [, mixed $from_encoding ] )
- 自动检测字符串编码，转成成指定编码，可能会影响效率。

```php
# GBK 参数可以省略
$outstr = mb_convert_encoding($instr,'UTF-8','GBK');
```


### PHP 防止代码注入，将 HTML 转换成实体，即转换成不可执行的字符

```php

$input_ok = htmlspecialchars($input);

```


### 简述高并发网站解决方案。
1. 前端优化（CND加速、建立独立图片服务器）
2. 服务端优化（页面静态化、并发处理[异步|多线程]、队列处理
3. 数据库优化（数据库缓存[Memcachaed|Redis]、读写分离、分库分表、分区
4. Web服务器优化（负载均衡、反向代理）


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


### PHP 获取 POST 上传的文件
- 前端

```html
<form action="upload_file.php" method="post" enctype="multipart/form-data">
    <label for="file">文件名：</label>
    <input type="file" name="file" id="file"><br>
    <input type="submit" name="submit" value="提交">
</form>

```

- 后端

```php
<?php
if ($_FILES["file"]["error"] > 0)
{
    echo "错误：: " . $_FILES["file"]["error"] . "<br>";
}
else
{
    echo "上传文件名: " . $_FILES["file"]["name"] . "<br>";
    echo "文件类型: " . $_FILES["file"]["type"] . "<br>";
    echo "文件大小: " . ($_FILES["file"]["size"] / 1024) . " kB<br>";
    echo "临时存储的位置: " . $_FILES["file"]["tmp_name"] . "<br>";
    
    move_uploaded_file($_FILES["file"]["tmp_name"], "upload/" . $_FILES["file"]["name"]);
    echo "文件存储在: " . "upload/" . $_FILES["file"]["name"];
    
}
```


### PHP 单引号和双引号的区别

- 单引号，不解析变量，所以单引号执行效率会更高。

- 双引号，会解析变量，所以双引号中的字符串可以包含变量。

### PHP 字符串和数组反转

```php
// 字符串
$s = "hello";
echo strrev($s);
// 数组
$a = array('a','b','c','d');
print_r(array_reverse($a));
```

### PHP debug 方法，即查看错误的方法

```php
// 允许显示错误
ini_set('display_errors', 1);
// 报告所有错误
error_reporting(-1);
```
