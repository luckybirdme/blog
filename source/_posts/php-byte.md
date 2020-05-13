---
title: php 进制数转换
date: 2020-05-09 00:40:02
tags: PHP
---

> 二进制

<!-- more -->

### base_convert ( string \$number , int \$frombase , int \$tobase ) 
- 任意进制字符串转为任意进制字符串
### bindec ( string $binary_string ) : number
- 二进制字符串转换为十进制
### decbin ( int $number ) : string
- 十进制转二进制字符串
### bin2hex ( string $str ) : string
- ASCII 字符串转十六进制


```php

// 十六进制数组
function hexArr($rev=false){
    $arr = array();
    for($i=0;$i<10;$i++){
        $arr[$i]=strval($i);
    }
    $arr[10]='a';
    $arr[11]='b';
    $arr[12]='c';
    $arr[13]='d';
    $arr[14]='e';
    $arr[15]='f';

    // 反转 key 和 val
    if($rev){
        $arr=array_flip($arr);    
    }
    return $arr;
}

// 十进制转任意进制
function myDec2Any($num,$to){
    $num = intval($num);
    $arr = hexArr();
    $h = '';
    while ( $num > 0) {
        // 余数
        $y = $num%$to;
        // 放末尾
        $h=$arr[$y].$h;
        // 求商
        $num = floor($num/$to);
    }
    
   
    return $h;
}

// 任意进制转十进制
function myAny2Dec($str,$from){
    // 转字符串
    $str = strval($str);
    // 求字符串长度
    $len = strlen($str);
    $dec = 0;
    
    // 字符串转为对应的数字
    $arr_rev = hexArr(true);

    // 遍历字符串
    for ($i=0; $i < $len; $i++) {
        // 以二进制为例 1111 = 2^3+2^2+2^1+2^0
        $dec=$dec + $arr_rev[$str[$i]] * pow($from, $len-1-$i);
    }

    return strval($dec);
}

function my_base_convert($str,$from,$to){
    return myDec2Any(myAny2Dec($str,$from),$to);   
}

$str='ad12';
var_dump(base_convert($str,16,2));
var_dump(my_base_convert($str,16,2));


function myBinDec($str){
    return intval(myDec2Any(myAny2Dec($str,2),10));
}

$str='110011';
var_dump(bindec($str));
var_dump(myBinDec($str));

function myBin2Hex($str){
    // 转字符串
    $str = strval($str);
    // 求字符串长度
    $len = strlen($str);
    $h='';
    // 遍历字符串
    for ($i=0; $i < $len; $i++) {
        // ord() 将字符转为 ASCII 值，即十进制
        // 十进制再转为十六进制
        $h=$h.myDec2Any(ord($str[$i]),16);
    }

    return $h;
}

$str='asdfsadfaf';
var_dump(bin2hex($str));
var_dump(myBin2Hex($str));

```


### base64_encode 转换
- 将原始字符串每三个字符分割，三个字符转为二进制，共有3*8=24位
- 将24位二进制每6位分割，即24/6=4组，为了凑够8位一个字节传输，每组前面加两个0，所以存储会比原始字符串大1/3
- 新分成的4组，转换成十进制，根据 base64 字典表，找到对应字符，即是 base64 编码
- 如果原始字符串不是三个倍数，余数0则不管
- 余数1，将最后一个字符转为8位二进制，后面追加4个0，凑够6的倍数12位，然后每6位分割，即12/6=2组，然后转换成十进制，找到 base64 编码，最后输出编码时，还要在末尾添加两个"="号
- 余数2，将最后两个字符转为16位二进制，后面追加2个0，凑够6的倍数18位，然后每6位分割，即18/=3组，然后转换成十进制，找到 base64 编码，最后输出编码时，还要在末尾添加一个"="号

```php

// 字符转为二进制，不够8位，自动在前面补0
function strToBin($str){
    $tmp = decbin(ord($str));
    // 少于8位，需要在前面添加0
    while (strlen($tmp)<8) {
        # code...
        $tmp='0'.$tmp;
    }
    return $tmp;
}
// 字符串每6为二进制分组，转换成十进制，根据 base64 字典，输出编码
function sixBinToBase64($bin){
    $base64='';
    if(empty($bin)){
        return $base64;
    }
    $base64_char="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
    for ($i=0; $i < strlen($bin);$i=$i+6) { 
        # code...
        // 每6位二进制分割
        $six_bin_tmp='';
        for ($j=0; $j < 6; $j++) { 
            # code...
            $six_bin_tmp=$six_bin_tmp.$bin[$i+$j];
        }
        // 转换成十进制，根据 base64 字典，输出编码
        $base64=$base64.$base64_char[bindec($six_bin_tmp)];
    }
    return $base64;
}
function my_base64_encode($str){
    $len = strlen($str);
    // 求出字符串除以3的倍数
    $str_thr = floor($len/3); 
    // 求出字符串除以3的余数
    $thr_last = $len%3;
    $base64='';

    // 遍历3倍数以内的字符串
    if ($str_thr) {
        // 转换成二进制
        $bin = '';
        for ($i=0; $i < $len-$thr_last; $i++) { 
            # code...
            // 转换十进制，再转为二进制
            $bin=$bin.strToBin($str[$i]);
        }
        $base64 = sixBinToBase64($bin);
        
    }
    

    // 如果余数等于0
    if($thr_last==0){
        return $base64;
    }
    // 等于1
    else if($thr_last==1){
        // 将最后一个字符转为二进制，共8位
        // 二进制位数后面追加4个0，筹够 6 的倍数 12
        $one_tmp = strToBin($str[$len-1]).'0000';
        // 编码后面追加两个"="号
        $base64 = $base64.sixBinToBase64($one_tmp).'==';
    }
    // 等于2
    else{
        // 将最后两个字符转为二进制，共16位
        // 二进制位数后面追加2个0，筹够 6 的倍数 18
        $one_tmp = strToBin($str[$len-2]).strToBin($str[$len-1]).'00';
        // 编码后面追加一个"="号
        $base64 = $base64.sixBinToBase64($one_tmp).'=';
    }
    return $base64;
}

$str="Masga";
var_dump(base64_encode($str));
var_dump(my_base64_encode($str));
var_dump(base64_encode($str)===my_base64_encode($str));

```