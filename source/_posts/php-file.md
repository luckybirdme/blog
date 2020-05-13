---
title: php 文件操作
date: 2020-05-09 07:45:48
tags: PHP
---

> file


<!-- more -->


### fopen, 文件读写支持多种操作模式

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


### fread ( resource $handle , int $length ) : string
- 从文件指针 handle 读取最多 length 个字节。 该函数在遇上以下几种情况时停止读取文件：读取了 length 个字节；到达了文件末尾（EOF）
- 返回值：返回所读取的字符串， 或者在失败时返回 FALSE。

```php

$myfile = fopen("test.txt", "r") or die("Unable to open file!");
echo fread($myfile,filesize("test.txt"));
fclose($myfile);

```

### fgets ( resource $handle [, int $length ] ) : string
- 从文件指针中读取一行。如果没有指定 length，则默认为 1K，从 PHP 4.3 开始，忽略掉 length 则行的长度被假定为 1024，将继续从流中读取数据直到行结束。如果文件中的大多数行都大于 8KB，则在脚本中指定最大行的长度在利用资源上更为有效。

```php
$handle = fopen('./file.txt', 'r');
// end of file, 测试文件指针是否到了文件结束的位置
while(!feof($handle)){
    echo fgets($handle,8012*2);
}
fclose($handle);
```


### fgetss ( resource $handle [, int $length [, string $allowable_tags ]] ) : string
- 和 fgets() 相同，只除了 fgetss() 尝试从读取的文本中去掉任何 HTML 和 PHP 标记。
- 一般用于读取 html 文件内容

### fpassthru ( resource $handle ) : int
- 将给定的文件指针从当前的位置，以流式读取到 EOF ，并把结果写到输出缓冲区，即标准输出通道
- 此函数将自动打印(echo)数据，因此无需使用变量获取数据。

```php
header("Content-Type:text/html;charset=utf-8"); 
$handle = fopen('./test.txt', 'r');
//将指针定位到1024字节处
fseek($handle, 1024);
// 将剩余文件内容输出到前端
fpassthru($handle);
```

### file ( string $filename [, int $flags = 0 [, resource $context ]] ) : array
- 把整个文件读入一个数组中。

```php
$a = file('./file.txt');
foreach($a as $line => $content){
  	echo 'line '.($line + 1).':'.$content;
}
```

### file_get_contents ( string $filename [, bool $use_include_path = false [, resource $context [, int $offset = -1 [, int $maxlen ]]]] ) : string
- 将整个文件读入一个字符串

```php
$homepage = file_get_contents('http://www.example.com/');
echo $homepage;

```


### readfile ( string $filename [, bool $use_include_path = false [, resource $context ]] ) : int 
- 与 fpassthru 类似，以流式读取文件，合适读取大文件，避免占满缓存区。
- 返回从文件中读入的字节数。如果出错返回 FALSE 并且除非是以 @readfile() 形式调用，否则会显示错误信息。


```php
// 强制下载图片文件
$file = 'monkey.gif';
if (file_exists($file)) {
    header('Content-Description: File Transfer');
    header('Content-Type: application/octet-stream');
    header('Content-Disposition: attachment; filename="'.basename($file).'"');
    header('Expires: 0');
    header('Cache-Control: must-revalidate');
    header('Pragma: public');
    header('Content-Length: ' . filesize($file));
    // 以流式读取图片文件，自动输出(echo)到前端
    @readfile($file);
    exit;
}
```


### parse_ini_file ( string $filename [, bool $process_sections = false [, int $scanner_mode = INI_SCANNER_NORMAL ]] ) : array
- 解析一个配置文件，ini 文件的结构和 php.ini 的相似，并以数组形式返回

```php
/*
; Comment
[personal information]
name = "King Arthur"
quest = To seek the holy grail
 
[more stuff]
Samuel Clemens = Mark Twain
Caryn Johnson = Whoopi Goldberg
*/

$file_array = parse_ini_file("holy_grail.ini");
print_r($file_array);
/*
Array
(
    [name] => King Arthur
    [quest] => To seek the Holy Grail
    [Samuel Clemens] => Mark Twain
    [Caryn Johnson] => Whoopi Goldberg
)
*/
```