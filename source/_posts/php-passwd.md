---
title: php 加密算法
date: 2020-04-28 07:49:06
tags: PHP
---

> 加密算法

<!-- more -->


## 前言
PHP加密方式主要包括：
- 散列加密，比如 md5、sha1、crypt、hash、，是不可逆的 
- 对称加密(DES、AES)，比如 urlencode，base64_encode，是可逆的
- 非对称加密，加密和解密的秘钥不是同一个，是最为安全的加密方法。


### md5
1. 定义
md5又称为报文摘要算法，将任意长度的信息作为输入值，换算成一个 128 位长度的"指纹信息"值来代表这个输入值。
2. 用途
md5算法是为数字签名应用程序而设计的，主要用于校验文件是否被改动，或者加密特殊文字，比如用户密码。
3. 案例

```php
$input_passwd = $_POST['passwd'];
// 第二个参数 raw 可选。规定十六进制或二进制输出格式：
// FALSE - 默认。32 字符十六进制数
// TRUE - 原始 16 字符二进制格式
$save_passwd = md5($input_passwd);
```


### sha1
1. 定义
Secure Hash Algorithm（安全哈希算法），根据美国安全散列算法 1 计算字符串的 sha1 散列值
2. 用途
与 md5 算法类似的用途，也是一种数字签名算法。
3. 案例

```php
$str = 'apple';
// 第二个参数可选
// FALSE - 默认。40 字符十六进制数
// TRUE - 原始 20 字符二进制格式
if (sha1($str) === 'd0be2dc421be4fcd0172e5afceea3970e2f3d940') {
    echo "Would you like a green or red apple?";
}
```

### urlencode 
1. 定义
将字符串中除了 -_. 之外的所有非字母数字字符都将被替换成百分号（%）后跟两位十六进制数，空格则编码为加号（+）。此编码与 WWW 表单 POST 数据的编码方式是一样的，同时与 application/x-www-form-urlencoded 的媒体类型编码方式一样。
2. 用途
主要是为了将特殊字符串的URL转码，方便进行传输，比如包含中文，解码方法是 urldecode
3. 案例

```php

$url="http://www.baidu.com/中文.rar";
$p_url = urlencode($url);
//http://www.baidu.com/%D6%D0%CE%C4.rar"
$o_url = urldecode($p_url);

```


### base64_encode
1. 定义
基于64个可打印字符来表示二进制数据的表示方法，编码后的数据比原始数据略长，为原来的 4/3。
2. 用途
设计此种编码是为了使二进制数据可以通过非纯 8-bit 的传输层传输，比如图片转换成文本。虽然图片可能直接传输，但也可以将它变成字符串直接放在源码里，而不需要浏览器再次从服务器下载。
3. 案例

```php

// 图片编码输出到前端
$img = 'test.jpg';
$base64_img = base64EncodeImage($img);
echo '<img src="' . $base64_img . '" />';

function base64EncodeImage ($image_file) {
  $base64_image = '';
  $image_info = getimagesize($image_file);
  $image_data = fread(fopen($image_file, 'r'), filesize($image_file));
  $base64_image = 'data:' . $image_info['mime'] . ';base64,' . chunk_split(base64_encode($image_data));
  return $base64_image;
}


// 后端解码保存图片
$input_base64 = $_POST['base64'];
get_base64_img($input_base64);

function get_base64_img($base64,$path = 'data/upload/sign/'){
    if (preg_match('/^(data:\s*image\/(\w+);base64,)/', $base64, $result)){
    	$type = $result[2];
       	$new_file = $path.time().".{$type}";

  		if (file_put_contents($new_file, base64_decode(str_replace($result[1], '', $base64)))){
    		return $new_file;
    	}
    }
    return false;
}
```




### crypt
1. 定义
返回一个基于标准 UNIX DES 算法或系统上其他可用的替代算法的散列字符串，具体算法也看PHP设置而确定。
crypt提供了以下的常量来标识是否支持某个算法（1表示支持，0表示不支持）：

```php
// 基于标准 DES 算法，使用 "./0-9A-Za-z" 字符中的两个字符作为盐值
if (CRYPT_STD_DES == 1) {
    echo 'Standard DES: ' . crypt('rasmuslerdorf', 'rl') . "\n";
}
// 扩展的DES 算法，以下划线_开头，后面紧接着4字节的迭代次数和4字节的盐值
if (CRYPT_EXT_DES == 1) {
    echo 'Extended DES: ' . crypt('rasmuslerdorf', '_J9..rasm') . "\n";
}
// md5 算法，使用一个以 $1$ 开始的 12 字符的字符串盐值。
if (CRYPT_MD5 == 1) {
    echo 'MD5:          ' . crypt('rasmuslerdorf', '$1$rasmusle$') . "\n";
}
// Blowfish 算法使用如下盐值：“$2a$”，一个两位 cost 参数，“$” 以及 64 位由 “./0-9A-Za-z” 中的字符组合而成的字符串
if (CRYPT_BLOWFISH == 1) {
    echo 'Blowfish:     ' . crypt('rasmuslerdorf', '$2a$07$usesomesillystringforsalt$') . "\n";
}
// SHA-256 算法使用一个以 $5$ 开头的 16 字符字符串盐值进行散列
if (CRYPT_SHA256 == 1) {
    echo 'SHA-256:      ' . crypt('rasmuslerdorf', '$5$rounds=5000$usesomesillystringforsalt$') . "\n";
}
//  SHA-512 算法使用一个以 $6$ 开头的 16 字符字符串盐值进行散列
if (CRYPT_SHA512 == 1) {
    echo 'SHA-512:      ' . crypt('rasmuslerdorf', '$6$rounds=5000$usesomesillystringforsalt$') . "\n";
}

/*
Standard DES: rl.3StKT.4T8M
Extended DES: _J9..rasmBYk8r9AiWNc
MD5:          $1$rasmusle$rISCgZzpwk3UhDidwXvin0
Blowfish:     $2a$07$usesomesillystringfore2uDLvp1Ii2e./U9C8sBjqp8I90dH6hi
SHA-256:      $5$rounds=5000$usesomesillystri$KqJWpanXZHKq2BOB43TSaYhEWsQ1Lr5QNyPCDH/Tp.6
SHA-512:      $6$rounds=5000$usesomesillystri$D4IrlXatmP7rx3P3InaxBeoomnAihCKRVQP22JZ6EY47Wc6BkroIuUUBOov1i.S5KPgErtP/EN5mcO.ChWQW21
*/

```

2. 用途
字符串加密，比如用户密码加密

3. 案例

```php
// 第二个参数是可选的盐值
// 自动生成盐值
$hashed_password = crypt('mypassword'); 

/* 你应当使用 crypt() 得到的完整结果作为盐值进行密码校验，以此来避免使用不同散列算法导致的问题。（如上所述，基于标准 DES 算法的密码散列使用 2 字符盐值，但是基于 MD5 算法的散列使用 12 个字符盐值。）*/
if (hash_equals($hashed_password, crypt($user_input, $hashed_password))) {
   echo "Password verified!";
}
```


### password_hash(>PHP 5.5)
1. 定义
使用足够强度的单向散列算法创建密码的散列（hash）。 password_hash() 兼容 crypt()。 所以， crypt() 创建的密码散列也可用于 password_hash()，官方也推荐使用这个。支持算法包括：
- PASSWORD_DEFAULT - 使用 bcrypt 算法 (PHP 5.5.0 默认)。 注意，该常量会随着 PHP 加入更新更高强度的算法而改变。 所以，使用此常量生成结果的长度将在未来有变化。 因此，数据库里储存结果的列可超过60个字符（最好是255个字符）。
- PASSWORD_BCRYPT - 使用 CRYPT_BLOWFISH 算法创建散列。 这会产生兼容使用 "$2y$" 的 crypt()。 结果将会是 60 个字符的字符串， 或者在失败时返回 FALSE。
- PASSWORD_ARGON2I - 使用 Argon2 散列算法创建散列。
2. 用途
字符串加密，比如用户密码加密

3. 案例

```php
// 使用默认算法散列密码，当前是 BCRYPT，并会产生 60 个字符的结果。
echo password_hash("rasmuslerdorf", PASSWORD_DEFAULT);

// 以上例程的输出类似于：
$2y$10$J55t9Dwi95K/qYg3/PQQD.sn5715ykfP80SmXBrYh3xm5aQMf2PHC

// 注意 password_hash() 返回的散列包含了算法、 cost 和盐值。 因此，所有需要的信息都包含内。使得验证函数不需要储存额外盐值等信息即可验证哈希。
$hash = '$2y$10$J55t9Dwi95K/qYg3/PQQD.sn5715ykfP80SmXBrYh3xm5aQMf2PHC';
if (password_verify('rasmuslerdorf', $hash)) {
    echo 'Password is valid!';
} else {
    echo 'Invalid password.';
}

// 自定义 salt 和 cost
$options = [
     //write your own code to generate a suitable salt
    'salt' => custom_function_for_salt(),
    // the default cost is 10
    'cost' => 12 
];
$hash = password_hash($password, PASSWORD_DEFAULT, $options);

// 更新 cost
if (password_needs_rehash($hash, PASSWORD_DEFAULT, ['cost' => 15])) {
    // cost change to 12
    $hash = password_hash($password, PASSWORD_DEFAULT, ['cost' => 15]);
    // don't forget to store the new hash!
}

```


### 特点
分类|urlencode|base64_encode
-|-|-
定义|特殊字符串转义|将二进制转换成十六进制字符
用途|特殊url转义|图片转换成文本传输


分类|md5|sha1
-|-|-
定义|散列(hash)函数|类似
用途|数字签名，文件是否被改动过|类似
区别|32个十六进制字符|40个十六进制字符
安全|通过收集，枚举，碰撞等方式破解|比md5安全一点
性能|输出字符串比sha1短，步骤也少|比md5慢约25%


分类|crypt|password_hash
-|-|-
定义|更多类型的哈希算法|兼容 crypt ，提供更强的哈希算法
用途|用户密码加密|类似
区别|需要手动保存盐值|自动提供和保存盐值，PHP后续版本也提倡此用法






