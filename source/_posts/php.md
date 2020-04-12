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


### 二位数组排序
```php

function mutiArraySort($key, $sort = SORT_DESC, $array) {
    $keyValue = [];
    foreach ($array as $k => $v) {
        $keyValue[$k] = $v[$key];
    }
    array_multisort($keyValue, $sort, $array);
    return $array;
}

$a=array(
        array('num'=>2,'name'=>'jack'),
        array('num'=>3,'name'=>'luse'),
        array('num'=>1,'name'=>'pony')

);

var_dump(mutiArraySort('num',SORT_DESC,$a));
var_dump(mutiArraySort('name',SORT_ASC,$a));
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


### 魔术方法
- \_\_construct, 构造函数，在实例化对象的时候执行
- \_\_destruct, 析构函数，在销毁对象后执行
- \_\_call, 调用一个未定义方法时默认调用
- \_\_get, 访问未定义的属性时默认调用
- \_\_toString, echo 输出类名时默认调用



### PHP 早期通过 mysql 扩展 API 操作 MySQL，后出现升级版 mysqli 
- mysql是非持继连接函数，mysql每次链接都会打开一个连接的进程。
- mysqli是永久连接函数，mysqli多次运行将使用同一连接进程，从而减少了服务器的开销。mysqli封装了诸如事务、prepared等一些高级操作，而且是面向对象的接口。

### PHP 官方默认使用 mysqli 操作 MySQL 数据库
```php
// mysql服务器主机地址，用户名，密码
$dbhost = 'localhost:3306';  
$dbuser = 'root';            
$dbpass = '123456';          
$conn = mysqli_connect($dbhost, $dbuser, $dbpass);
if(! $conn )
{
    die('mysqli_connect : ' . mysqli_error($conn));
}
// 设置编码
mysqli_query($conn , "set names utf8");
 
// 选择 DB
mysqli_select_db( $conn, 'mydatabase' );

// 执行 SQL
$sql = 'SELECT * FROM mytable LIMIT 10';
$retval = mysqli_query( $conn, $sql );
if(! $retval )
{
    die('mysqli_query : ' . mysqli_error($conn));
}

// 获取数据
while($row = mysqli_fetch_array($retval, MYSQLI_ASSOC))
{
    var_dump($row);
}
// 关闭链接
mysqli_close($conn);
```


### PDO, 全称 PHP data object, PHP 操作数据库的抽象 API
- PHP 基于抽象 API 方便切换到不同类型的数据库。 mysql 和 mysqli 只支持 MySQL 数据库。
- mysql, mysqli, PDO 底层都是调用 mysqlnd (MySQL Native Driver) 跟 mysql-server 进行网络协议交互。


### mysqli 和 PDO 支持特殊字符转义避免 SQL 注入。
```php
// mysqli, "manual" escaping
$username = mysqli_real_escape_string($_GET['username']);
$mysqli->query("SELECT * FROM users WHERE username = '$username'");

// PDO, "manual" escaping
$username = PDO::quote($_GET['username']);
$pdo->query("SELECT * FROM users WHERE username = $username");
        
```


### mysqli 和 PDO 都支持更高效更安全的 prepared statement 参数绑定。
预处理  prepared statement  ，首先将 SQL 进行语义分析，然后再传递变量，而这时候的变量只当做数据来处理，即使变量包含了SQL，也不会当做SQL来执行，从而在根本上避免了输入变量包含SQL注入的风险。

```php

// mysqli
$query = $mysqli->prepare('
    SELECT * FROM users
    WHERE username = ?
    AND email = ?
    AND last_login > ?');
// sss, 变量类型和数量，3 个字符串
// 后面参数按照顺序替换到 SQL 中
$query->bind_param('sss', 'test', $mail, time() - 3600);
$query->execute();

// pdo
$pdo->prepare('
    SELECT * FROM users
    WHERE username = :username
    AND email = :email
    AND last_login > :last_login');
     
// 根据关键字替换变量
$pdo->execute(array(':username' => 'test', ':email' => $mail, ':last_login' => time() - 3600));

```


### PHP 通过事物操作 MySQL
```php
// mysql服务器主机地址，用户名，密码
$dbhost = 'localhost:3306';  
$dbuser = 'root';            
$dbpass = '123456';          
$conn = mysqli_connect($dbhost, $dbuser, $dbpass);
if(! $conn )
{
    die('mysqli_connect : ' . mysqli_error($conn));
}

// 设置编码
mysqli_query($conn, "set names utf8");
// 设置当前 session 不自动提交，MySQL 默认自动提交
mysqli_query($conn, "set session autocommit=0"); 

// 选择 DB
mysqli_select_db( $conn, 'mydatabase' );

// 开始事务定义
mysqli_begin_transaction($conn);            
 
if(!mysqli_query($conn, "insert into mytable (id) values(8)"))
{
    // 判断当执行失败时回滚
    mysqli_query($conn, "ROLLBACK");     
}
 
// 提交事务
mysqli_commit($conn);         
// 关闭连接   
mysqli_close($conn);
```


