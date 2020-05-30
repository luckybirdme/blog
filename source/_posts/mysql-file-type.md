---
title: MySQL 数据文件
date: 2020-05-23 12:35:43
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->


### 数据文件目录

##### 1. 查看所在目录
```sql
mysql> show variables like "%datadir%";
+---------------+-------------------------------------+
| Variable_name | Value                               |
+---------------+-------------------------------------+
| datadir       | D:\wamp\bin\mysql\mysql5.7.14\data\ |
+---------------+-------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

##### 2. 主要数据文件

```sh
2020/05/23  09:17                 6 Mycomputer-NB1.slow.log
2020/05/17  23:23               371 ib_buffer_pool
2020/05/23  12:32         5,242,880 ib_logfile0
2019/04/14  18:55         5,242,880 ib_logfile1
2020/05/23  12:32        12,582,912 ibdata1
2020/05/23  09:17        12,582,912 ibtmp1
2019/04/14  19:17    <DIR>          mysql
2019/04/14  18:55    <DIR>          performance_schema
2019/04/14  18:55    <DIR>          sys
2020/05/23  12:29    <DIR>          test_file_db
```

![](/img/2020/mysql_file_0.png)
![](/img/2020/mysql_file_00.png)



### slow.log

##### 1. 由来
记录在 MySQL 中响应时间超过阀值的语句，具体指运行时间超过 long_query_time 值，默认 10 秒

##### 2. 作用
记录慢日志，用于SQL调优

##### 3. 设置
- long_query_time=10.0, 设置慢日志阈值, 单位秒
- slow_query_log=ON, 设置开启慢日志
- slow_query_log_file, 指定慢日志文件

```sql
mysql> show variables like "%long_query_time%" \G
*************************** 1. row ***************************
Variable_name: long_query_time
        Value: 10.000000
1 row in set, 1 warning (0.00 sec)
mysql> show variables like "%slow_query_log%" \G
*************************** 1. row ***************************
Variable_name: slow_query_log
        Value: ON
*************************** 2. row ***************************
Variable_name: slow_query_log_file
        Value: D:\wamp\bin\mysql\mysql5.7.14\data\Mycomputer-NB1-slow.log
2 rows in set, 1 warning (0.00 sec)

```



### ib_buffer_pool 

##### 1. 由来
InnoDB 引擎维护了一个缓存区域叫做 Buffer Pool，将热数据和索引缓存在内存中，也可以导出到缓存文件 ib_buffer_pool

##### 2. 作用
Buffer Pool 可以加速数据的读写，如果 Buffer Pool 越大，那么 MySQL 就越像一个内存数据库，所以 Buffer Pool 是为了提升 MySQL 的性能。

##### 3. 设置
- innodb_buffer_pool_dump_at_shutdown=ON, 关闭 MySQL 的时候，自动导出热数据到缓存文件 ib_buffer_pool
- innodb_buffer_pool_dump_now=ON, 立即导出热数据到缓存文件 ib_buffer_pool
- innodb_buffer_pool_load_at_startup = ON, 在启动时把缓存文件 ib_buffer_pool 的热数据加载到内存中
- innodb_buffer_pool_load_now = ON, 立即将缓存文件 ib_buffer_pool 的热数据加载到内存中

```sql
mysql> show variables like "%buffer_pool%";
+-------------------------------------+----------------+
| Variable_name                       | Value          |
+-------------------------------------+----------------+
| innodb_buffer_pool_chunk_size       | 16777216       |
| innodb_buffer_pool_dump_at_shutdown | ON             |
| innodb_buffer_pool_dump_now         | OFF            |
| innodb_buffer_pool_dump_pct         | 25             |
| innodb_buffer_pool_filename         | ib_buffer_pool |
| innodb_buffer_pool_instances        | 1              |
| innodb_buffer_pool_load_abort       | OFF            |
| innodb_buffer_pool_load_at_startup  | ON             |
| innodb_buffer_pool_load_now         | OFF            |
| innodb_buffer_pool_size             | 16777216       |
+-------------------------------------+----------------+
```


### ib_logfile

##### 1. 由来
InnoDB 引擎支持事物机制，每个事物操作都会记录在 ib_logfile 文件中，是 InnoDB 事物操作特有的数据文件。
特此说明，ib_logfile 是引擎层产生的日志，主要是事物提交前落地到磁盘，而 binlog 是 MySQL server 层的二进制日志，是事物提交后的数据记录，无论引擎是否支持事物，都会有 binlog, 包括 MyISAM . 

##### 2. 作用
InnoDB 在事物提交前会先写到本地磁盘即 ib_logfile, 因此可以作为 MySQL 异常宕机后启动时，重做事物。
binlog 主要是恢复数据库的数据，或者在主从数据库之间的同步时使用。

##### 3. 设置
- innodb_log_files_in_group=2, 表示两个 ib_logfile0,ib_logfile1 文件，采取循环写的方式，写完第一个文件再到第二个，然后再到第一个，以此循环。


```sql

mysql> show variables like "%innodb_log_files_in_group%" ;
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| innodb_log_files_in_group | 2     |
+---------------------------+-------+
1 row in set, 1 warning (0.01 sec)

```

![](/img/2020/mysql_file_1.png)

![](/img/2020/mysql_file_one.png)



### ibdata

##### 1. 由来
共享表空间的表数据文件，MySQL 5.6 后可通过 innodb_file_per_table = ON 设置每个表单独一个表空间文件
如果 innodb_undo_tablespaces = 0, 此文件还会包括 undo logs 的内容.

##### 2. 作用
将所有表空间保存到一起，方便统一管理和迁移。
undo logs 用于事物操作出现错误时，回滚数据，保证事务的原子性。


##### 3. 设置

- innodb_data_file_path=ibdata1:10M:autoextend, 文件最大10M，超过后自动增长
- innodb_data_file_path=ibdata1:10M;/disk2/ibdata2:50M:autoextend, 支持多个表空间文件，而且不同表空间文件可以额存放到不同磁盘

```sql
mysql> show variables like "%innodb_data_file_path%" ;
+----------------------------+------------------------+
| Variable_name              | Value                  |
+----------------------------+------------------------+
| innodb_data_file_path      | ibdata1:10M:autoextend |
+----------------------------+------------------------+
1 rows in set, 1 warning (0.00 sec)
```


![](/img/2020/mysql_file_2.png)


### ibtmp

##### 1. 由来
跟 ibdata 类似，只不过是临时表的共享表空间

##### 2. 作用
通过 explain 分析判断是否使用临时表，参数 Extra: Using temporary;

```sql
mysql> explain select distinct scode,insert_at  from main_stock order by insert_at limit 1 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: main_stock
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 30216
     filtered: 100.00
        Extra: Using temporary; Using filesort
1 row in set, 1 warning (0.00 sec)

```

临时表主要可能出现在以下场景用到
- UNION查询；
- 用到TEMPTABLE视图；
- ORDER BY和GROUP BY的子句不一样时；
- DISTINCT查询并且加上ORDER BY时；
- FROM中包括子查询；



##### 3. 设置

- 跟 innodb_data_file_path 参数类似

```sql
mysql> show variables like "%innodb_temp_data_file_path%" ;
+----------------------------+-----------------------+
| Variable_name              | Value                 |
+----------------------------+-----------------------+
| innodb_temp_data_file_path | ibtmp1:12M:autoextend |
+----------------------------+-----------------------+
1 row in set, 1 warning (0.00 sec)

```


### 系统自带表的表空间文件

##### 1. 由来
ibdata 是共享表空间文件，当设置了 innodb_file_per_table = ON, 每个表单独一个表空间文件，所以每个数据库会有一个目录，目录下是数据库每个表的表空间问价。
MySQL 自带的数据库，包括 mysql, performance_schema, sys 等，这些数据库都有自己文件目录和单独表空间。

##### 2. 作用

- mysql库, 包含权限配置，事件，存储引擎状态，主从信息，日志，时区信息，用户权限配置等
- performance_schema, 通过事件机制将mysql服务的运行时状态采集并存储在 performace_schema 数据库。
- sys, 此数据库的数据主要源自 performance_schema 数据库，其目标是把查询performance_schema 的复杂度降低，更好地利用这个库里的数据，更快地了解 MySQL 的运行情况。
- information_schema, 用于存储数据库元数据，例如数据库名、表名、列的数据类型、访问权限等，此表实际上是视图，而不是基本表，因此，文件系统上没有与之相关的文件。


##### 3. 设置

- innodb_file_per_table=ON, 表示 InnoDB 每个数据库表单独保存表空间文件。
```sql
mysql> show variables like '%innodb_file_per_table%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set, 1 warning (0.01 sec)

```



### 用户创建表的表空间文件

##### 1. 由来
test_file_db 是用户创建的数据库，因为是独享表空间，所以存在这个目录，目录下里面包括每个表的表空间文件。

```sh
2020/05/23  10:20                65 db.opt
2020/05/23  12:00             8,556 test_file_tb.frm
2020/05/23  12:32                 7 test_file_tb.MYD
2020/05/23  12:00             1,024 test_file_tb.MYI
2020/05/23  12:29             8,586 tt_ff_tb.frm
2020/05/23  12:32            98,304 tt_ff_tb.ibd
```


##### 2. db.opt
此文件保存了数据库创建时指定的默认字符串，删除掉的话，会取默认的全局字符集，所以影响不大。

```sh
default-character-set=latin1
default-collation=latin1_swedish_ci
```

##### 3. frm 后缀文件
表结构文件，这与存储引擎无关，每个表格都会。

##### 4. MYD 和 MYI 后缀文件
- 这两种文件是 MyISAM 引擎特有的文件
- MYI 索引信息文件，可知 MyISAM 引擎的索引和数据是分开保存的。
- MYD 数据信息文件，此文件只在采用独立表存储模式才会有，innodb_file_per_table = ON


```sql
mysql> show create table test_file_tb;
+--------------+----------------------------------------------------------------
| Table        | Create Table
+--------------+----------------------------------------------------------------
| test_file_tb | CREATE TABLE `test_file_tb` (
  `id` int(11) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=latin1 |
+--------------+----------------------------------------------------------------
1 row in set (0.00 sec)
```

##### 5. ibd 后缀文件
- 此类型文件是 InnoDB 特有的，索引和数据都保存在一个文件中


```sql

mysql> show create table tt_ff_tb;
+----------+----------------------------------------------------------
| Table    | Create Table
+----------+----------------------------------------------------------
| tt_ff_tb | CREATE TABLE `tt_ff_tb` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8 |
+----------+----------------------------------------------------------
1 row in set (0.00 sec)
```


### 共享表空间和独立表空间

##### 1. 区分
- innodb_file_per_table = ON, 独立表空间，每个表格的数据单独一个文件保存

##### 2. 共享表空间

- 优点
 - 表空间可以分成多个文件存放到各个磁盘，表的大小不受磁盘大小的限制。
 - 所有表格数据文件放在一起方便管理。

- 缺点
 - 所有表格数据存放到一个文件，虽然可以把一个大文件分成多个小文件，但是多个表及索引在表空间中混合存储，当数据量非常大的时候，表做了大量删除操作后表空间中将会有大量的空隙，特别是对于统计分析，对于经常删除操作的这类应用最不适合用共享表空间。

 - 共享表空间分配后不能回缩，当出现临时建索引或是创建一个临时表的操作表空间扩大后，就是删除相关的表也没办法回缩那部分空间了，可以理解为表空间才使用10M，但是操作系统显示mysql的表空间为10G，进行数据库的冷备很慢


##### 3. 独立表空间
- 优点
 - 每个表都有自已独立的表空间，数据存储清晰
 - 可以实现单表在不同的数据库中移动，以及恢复。
 - 空间可以回收。

- 缺点
 - 单表增加过大，当单表占用空间过大时，存储空间不足，只能从操作系统层面思考解决方法。
 - 进行 delete 操作后，特别是有Text和BLOB，会留下许多的数据空洞，也称为碎片，不仅占用空间，也影响搜索，需要借助 optimize table my_tb 命令来整理碎片。




