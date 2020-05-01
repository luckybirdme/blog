---
title: php 链接数据库
date: 2020-04-29 07:16:47
tags: PHP
---

> MySQL

<!-- more -->





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