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





### require 和 include 区别
- require 找不到文件或者返回严重错误之后脚本就会终止执行
- include 无法找到文件时，PHP 会继续执行下去


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



### PHP 函数 split 和 explode 区别
- explode ( string \$delimiter , string \$string ) : array, 是以指定字符来分割字符串，并返回数组
- split 是通过正则匹配来分割字符串。

### PHP 函数 join 和 implode 区别
- implode ( string \$glue , array $pieces ) : string, 用指定字符将数组元素拼接在一起，并返回字符串。
- join 是 implode 的别名，所以是一样的。


### PHP7新特性
- 属性类型限定和提示，类似强类型语言 Java
- 底层编译运行性能优化，包括内存使用，引擎解析过程等