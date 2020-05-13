---
title: mysql 数据类型
date: 2020-05-08 07:23:56
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->



### 整型
类型	|大小(byte)|范围（有符号）|范围（无符号）|用途
-|-|-|-|-
TINYINT	|1 = 2^8	|(-128，127)|	(0，255)|	小整数值
SMALLINT|2 = 2^16	|(-32 768，32 767)|	(0，65 535)	|中等整数值
MEDIUMINT	|3 = 2^24 |(-8 388 608，8 388 607)	|(0，16 777 215)	|大整数值
INT|4 	= 2^32|(-2 147 483 648，2 147 483 647)	|(0，4 294 967 295)	|大整数值
BIGINT	|8 =2^64	|(-9,223,372,036,854,775,808，9 223 372 036 854 775 807)	|(0，18 446 744 073 709 551 615)	|极大整数值
FLOAT	|4 = 2^32 |	(-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38)|	0，(1.175 494 351 E-38，3.402 823 466 E+38)	|单精度浮点数值
DOUBLE	|8 =2^64	|(-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)	|0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)	|双精度浮点数值


### int(10) 和 int(11)
- int 占用 4 个字节(byte)，一个字节 8 bit，那么可以代表 2^32 个状态，因此对于正整数(无符号)，可保存范围是 0-2^32 的数据
- int(n)，这里的 n 是显示的长度，不是保存数据的长度，因此 int(10) 和 int(11) 只是显示长度不一样，int 字段依然是占用 4 个字节，保存 0-2^32 的数据。


### 字符串

类型	|大小(字符数量)|	用途
-|-|-
CHAR|	0-255 < 2^8 |	定长字符串
VARCHAR	|0-65535 < 2^16-2 |	变长字符串，可以有默认值
BLOB/TEXT	|0-65535 < 2^16-2 |	二进制形式/字符串形式的文本数据
MEDIUMBLOB/MEDIUMTEXT|	0-16 777 215 < 2^24-3 |	中等数据
LONGBLOB/LONGTEXT|	0-4 294 967 295 < 2^32-4 |	极大数据


### char 和 varchar 

- 区别

区别|char|varchar
-|-|-
字符长度范围|0-255|0-65535|
数据存放方式|固定长度保存，太短会在后面补齐空格|实际长度保存，前缀 overhead 添加1或2个字节，记录字符长度。
索引|自动剔除字符后面空格进行检索|保留字符后面的空格进行检索

1. char(10) 和 varchar(10) 的意思是保存最多 10 个字符，如果是utf8编码，每个字符3个字节，那么最长占用了10*3=30个字节。
2. MySQL 限制每行数据最长 65535 字节，即每个字段设置数据长度的总值不能超过 65535，如果表中只有一个 varchar 字段，而且是utf8编码，那么最长保存 65535/3=21845个字符。
3. 对于 latin1 字符集(ISO-8859-1的别名)，一个字符占用一个字节，所以表中只有一个 varchar 字段时，最大字符长度可设置 65535。
4. 如果字符大于255(2^8=8bit=1byte)，varchar 会保存字符数据时会在 overhead 添加一个字节记录字符长度，如果大于255，则是添加两个字节。

```mysql
mysql> create table t7 (col_one varchar(20000) default '') charset=utf8;
Query OK, 0 rows affected (0.19 sec)

mysql> create table t8 (col_one varchar(30000) default '') charset=utf8;
ERROR 1074 (42000): Column length too big for column 'col_one' (max = 21845); use BLOB or TEXT instead

mysql> create table t8 (col_one varchar(30000) default '') charset=latin1;
Query OK, 0 rows affected (0.22 sec)

mysql> create table t8 (col_one varchar(70000) default '') charset=latin1;
ERROR 1074 (42000): Column length too big for column 'col_one' (max = 65535); use BLOB or TEXT instead
```

- 使用建议

1. char 是固定长度，存取效率稍高于 varchar，但在 MySQL5.7 的 InnoDB 引擎，其实性能相差不大。
2. varchar 以实际字符长度保存，如果平均字符长度小于255，varchar 占用的空间会比 char 更小。
3. 建议尽量使用 varchar，除非是明确数据字符长度是固定的，比如手机号码，考虑使用 char



### varchar 和 text


- 区别

区别|varchar|text
-|-|-
默认值|可设置|不能有默认值
保存字符长度|必须指定|不需要设置
索引值长度|默认使用列值全部内容|必须设置列值前缀长度
数据存放方式|保存在当前行，被 MySQL 最长行字节 65535 限制|只占用当前行 9 到 12 个字节，实际数据保存到另外行

1. 索引建立会占用空间，索引值长度也会影响性能，text 字段建立索引时，需要指定列值前缀长度，作为最终的索引值长度。varchar 字段默认是列值全部作为索引长度，也可设置指定长度。
2. varchar 字段的数据保存在当前行，所以检索效率会比 text 数据更高。而 text 的数据是保存到另外行或者页的，因此可以保存更多的数据。

```mysql

mysql> create table t8 (col_one varchar(30000) default '') charset=utf8;
ERROR 1074 (42000): Column length too big for column 'col_one' (max = 21845); use BLOB or TEXT instead
mysql> create table t8 (col_one text) charset=utf8;
Query OK, 0 rows affected (0.01 sec)

mysql> alter table t8 add index col_one(col_one);
ERROR 1170 (42000): BLOB/TEXT column 'col_one' used in key specification without a key length
mysql> alter table t8 add index col_one(col_one(10));
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

```


- 使用建议
1. 如果需要默认值，请使用 varchar
2. 如果行数据没有超过限制 MySQL 限制的 65535，尽量使用检索更高效的 varchar
3. 如果行数据超过了 65535 限制，则使用 MEDIUMTEXT, LONGTEXT



### 时间
类型	|大小(byte)|范围	|格式|	用途
-|-|-|-|-
DATE|3|	1000-01-01/9999-12-31	|YYYY-MM-DD	|日期值
TIME|3	|-838:59:59/838:59:59	|HH:MM:SS|	时间值或持续时间
TIMESTAMP|	4	|1970-01-01 00:00:00/2038-01-19 03:14:07 |YYYY-MM-DD HH:MM:SS|混合日期和时间值
DATETIME	|8	|1000-01-01 00:00:00/9999-12-31 23:59:59	|YYYY-MM-DD HH:MM:SS	|混合日期和时间值


### TIMESTAMP 和 DATETIME

- 区别

区别|TIMESTAMP |  DATETIME 
-|-|-
占用空间|4个字节|8个字节
时间范围|1970-01-01 00:00:00/2038-01-19 03:14:07|1000-01-01 00:00:00/9999-12-31 23:59:59
存储方式|插入的时间从当前时区转化为世界标准时间(UTC)进行存储|与时区无关，基本上是原样输入和输出

- 使用建议

1. 考虑到以后时间范围会超过2038年，建议使用 DATETIME
2. 如果是跨时区的业务场景，使用 TIMESTAMP 会更为合适
3. MySQL 5.6.5 版本开始，DATETIME 类型才支持 DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP