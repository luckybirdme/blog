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


### 单实例模式：防止重复实例化，避免大量的new操作，减少消耗系统和内存的资源，使得有且仅有一个实例对象

```php

# 单实例的类
class Singleton
{
    //创建静态私有的变量保存该类对象
    private static $instance;

    //防止使用new直接创建对象
    private function __construct(){}

    //防止使用clone克隆对象
    private function __clone(){}

    public static function getInstance()
    {
        //判断$instance是否是Singleton的对象，不是则创建
        if (!self::$instance instanceof self) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    public function show_name()
    {
        echo "Singleton";
    }
}

$sing = Singleton::getInstance();
$sing->show_name();
$sing2 = new Singleton(); 
//Fatal error: Uncaught Error: Call to private Singleton::__construct() from invalid context in
$sing3 = clone $sing; 
//Fatal error: Uncaught Error: Call to private Singleton::__clone() from context

```


### 工厂模式：用工厂方法代替new操作的一种模式，如果需要更改所实例化的类名，只需在工厂方法内修改，不需逐一寻找代码中具体实例化的地方

```php
/**
 * 测试类一
  */
class demo1
{
    //定义一个test1方法
    public function test1()
    {
        echo '这是demo1类的test1方法'.PHP_EOL;
    }
}
/**
 * 测试类二
  */
class demo2
{
    //定义一个test2方法
    public function test2()
    {
        echo '这是demo2类的test2方法'.PHP_EOL;
    }
}
/**
 * 工厂类
 */
class Factoty
{
    // 根据传参类名，创建对应的对象
    static function createObject($className)
    {
        return new $className();
    }
}
/**
 * 通过传类名，调用工厂类里面的创建对象方法
 */
$demo = Factoty::createObject('demo1');
$demo->test1();             //输出这是demo1类的test1方法
$demo = Factoty::createObject('demo2');
$demo->test2();            //输出这是demo2类的test2方法

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


### PHP 属性的限制
1. private : 私有成员, 在类的内部才可以访问。
2. protected : 保护成员，该类内部和继承类中可以访问。
3. public : 公共成员，完全公开，没有访问限制。


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