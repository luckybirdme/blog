---
title: MySQL 知识点
date: 2020-04-04 17:41:09
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->

### 整型
类型	|大小(byte)|范围（有符号）|范围（无符号）|用途
-|-|-|-|-
TINYINT	|1 	|(-128，127)|	(0，255)|	小整数值
SMALLINT|2 	|(-32 768，32 767)|	(0，65 535)	|中等整数值
INT|4 	|(-2 147 483 648，2 147 483 647)	|(0，4 294 967 295)	|大整数值
BIGINT	|8 	|(-9,223,372,036,854,775,808，9 223 372 036 854 775 807)	|(0，18 446 744 073 709 551 615)	|极大整数值
FLOAT	|4 |	(-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38)|	0，(1.175 494 351 E-38，3.402 823 466 E+38)	|单精度浮点数值

### 时间
类型	|大小(byte)|范围	|格式|	用途
-|-|-|-|-
DATE|3|	1000-01-01/9999-12-31	|YYYY-MM-DD	|日期值
DATETIME	|8	|1000-01-01 00:00:00/9999-12-31 23:59:59	|YYYY-MM-DD HH:MM:SS	|混合日期和时间值
TIMESTAMP|	4	|1970-01-01 00:00:00 到现在的秒数|YYYYMMDD HHMMSS	|混合日期和时间值，时间戳

### 字符串

类型	|大小(bytes)|	用途
-|-|-
CHAR|	0-255 |	定长字符串
VARCHAR	|0-65535 |	变长字符串，可以有默认值
BLOB	|0-65535 |	二进制形式的长文本数据
TEXT	|0-65535 |	长文本数据，不能有默认值
MEDIUMBLOB|	0-16 777 215 |	二进制形式的中等长度文本数据
MEDIUMTEXT|	0-16 777 215 |	中等长度文本数据
LONGBLOB|	0-4 294 967 295 |	二进制形式的极大文本数据
LONGTEXT|	0-4 294 967 295 |	极大文本数据

使用总结
- 经常变化的字段用varchar
- 知道固定长度的用char
- 尽量用varchar
- 超过255字符的只能用varchar或者text
- 能用varchar的地方不用text
- 能用整型就不用字符型，因为前者查询效率更高



### 事物操作的特点
1. 原子性：一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没有执行过一样。

2. 一致性：事务执行结束后，数据库的完整性约束没有被破坏，事务执行的前后都是合法的数据状态。实体完整性，比如行的主键存在且唯一；列完整性，比如如字段的类型、大小、长度要符合要求。

3. 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。

4. 持久性：事务一旦提交完成，它对数据库的改变就应该是永久性的，接下来的其他操作或故障不应该对其有任何影响，比如 MySQL 宕机 。


### 事物隔离的问题

- 脏读：事物A插入数据之后未提交，此时事物B读到这些未提交数据，之后事物A回滚了，事物B再去读这些数据时，发现不见了，即之前事物B读到的数据不存在了，是脏数据。
- 不可重复读：事物B在第一次查询只有一条记录，这是事物A执行了插入数据，并且提交了；然后事物B第二次查询时发现多了一条数据，跟第一次查询结果不一样了，即同一事物中，不可以重复查询了。
- 幻读：事物A查询发现没有id=3的数据，这时事物B执行了插入id=3的数据，并且提交了；然后事物A执行同样id=3的数据插入，发现报错了，提示已经存在id=3的数据了，但是事物A查询却看不到id=3的数据，这是因为同一事物中读取的数据从开始之后，到提交之前都不变，即使被其他事物修改并且提交了，当前事物依然看不到这些新增的数据，


### 隔离级别以及产生的问题

隔离解别|	脏读	|不可重复读|	幻读
-|-|-|-
read-uncommitted/可读取未提交|	Y	|Y	|Y
read-committed/只读取已提交|	N|	Y	|Y|
repeatable-read/同一个事物可重复读(默认级别)|	N	|N|	Y
Serializable/串行化(锁竞争大性能差)|	N|	N|	N


### MySQL 事物默认自动提交，通过以下命令查看和修改，修改全局 global 级别会对所有登录用户生效，也可以仅对当前 session 修改。

```mysql
mysql> show session variables like 'autocommit'; 
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.00 sec)

mysql> set session autocommit = 0;
Query OK, 0 rows affected (0.00 sec)

mysql> show session variables like 'autocommit'; 
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | OFF   |
+---------------+-------+
1 row in set (0.00 sec)


mysql> show global variables like 'autocommit'; 
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.00 sec)

mysql> set global autocommit = 0;
Query OK, 0 rows affected (0.00 sec)

mysql> show global variables like 'autocommit'; 
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

```

### MySQL 隔离级别默认是 repeatable-read，跟 autocommit 一样，分全局 global 和当前 session 
```mysql

mysql> select @@global.tx_isolation,@@tx_isolation;
+-----------------------+-----------------+
| @@global.tx_isolation | @@tx_isolation  |
+-----------------------+-----------------+
| REPEATABLE-READ       | REPEATABLE-READ |
+-----------------------+-----------------+

mysql> set global tx_isolation='read-uncommitted';  
Query OK, 0 rows affected (0.00 sec)

mysql> set session tx_isolation='read-uncommitted';
Query OK, 0 rows affected (0.00 sec)

mysql> select @@global.tx_isolation,@@tx_isolation;
+-----------------------+------------------+
| @@global.tx_isolation | @@tx_isolation   |
+-----------------------+------------------+
| READ-UNCOMMITTED      | READ-UNCOMMITTED |
+-----------------------+------------------+
1 row in set (0.00 sec)

```


### MySQL 的事物操作主要包括三个方法
- BEGIN(或者 START TRANSACTION), 开始一个事物
- ROLLBACK, 回滚一个事物
- COMMIT, 提交一个事物

```mysql

mysql>  select * from mytable;
+------+
| age   |
+------+
| 4    |
+------+

mysql> begin;  
Query OK, 0 rows affected (0.00 sec)
 
mysql> insert into mytable (age) value(5);
Query OK, 1 rows affected (0.01 sec)

mysql> commit; 
Query OK, 0 rows affected (0.01 sec)

mysql>  select * from mytable;
+------+
| age   |
+------+
| 4    |
| 5    |
+------+

mysql> begin;   
Query OK, 0 rows affected (0.00 sec)
 
mysql>  insert into mytable(age) values(6);
Query OK, 1 rows affected (0.00 sec)
 
mysql> rollback; 
Query OK, 0 rows affected (0.00 sec)

mysql>  select * from mytable;
+------+
| age   |
+------+
| 4    |
| 5    |
+------+

```


### union 和 union all
- union 去掉重复数据
- union all 包括重复数据

### MySQL 临时表用于保存临时数据，show tables 下无法查看，而且只在当前连接操作，当关闭连接时，Mysql会自动删除表并释放所有空间。

```mysql
mysql> create temporary table user_temp(name char(100) default '');
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
Empty set (0.00 sec)

mysql> insert into user_temp(name) values('jack');
Query OK, 1 row affected (0.00 sec)

mysql> select * from user_temp;
+------+
| name |
+------+
| jack |
+------+
1 row in set (0.00 sec)
```

### distinct 和 group by 功能类似，如下是相同的输出
```mysql
mysql> SELECT DISTINCT last_name, first_name
    -> FROM person_tbl;

mysql> SELECT last_name, first_name
    -> FROM person_tbl
    -> GROUP BY last_name, first_name;
```

### mysqldump 方式包括导出表格结构和数据行，其中数据行采用 insert 语句
```sh
$ mysqldump -u root -p database_name table_name > dump.sql

$ mysql -u root -p other_database_name < dump.sql
```

### 更高效的数据导出和导入方式
1. select into outfile 文件一行内容对应表格一行数据
2. load data infile 比 insert 语句更高效
```mysql

mysql> SELECT * FROM stock INTO OUTFILE 'D:/wamp/tmp/stock.txt';
Query OK, 898 rows affected (0.01 sec)

mysql> create table stock_back like stock;
Query OK, 0 rows affected (0.01 sec)

mysql> load data infile  'D:/wamp/tmp/stock.txt' into table stock_back;
Query OK, 898 rows affected (0.02 sec)
Records: 898  Deleted: 0  Skipped: 0  Warnings: 0
```
