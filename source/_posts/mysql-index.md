---
title: MySQL 性能优化
date: 2020-04-05 15:13:20
tags: MySQL
---

> 本文基于 MySQL 5.7 

<!-- more -->


## 一. MySQL 逻辑架构
![](http://qiniucdn.luckybird.me/blog/img/2020/MySQL.png)

### 1. 连接层(Client Connectors)：
最上层是一些客户端和连接服务，所包含的服务并不是MySQL所独有的技术。它们都是服务于C/S程序或者是这些程序所需要的 ：连接处理，身份验证，安全性等等。

### 2. 服务层(MySQL server)：
主要完成大多数的核心服务功能，如SQL 接口，并完成缓存的查询，SQL 的分析和优化及部分内置函数的执行。所有跨存储引擎的功能也在这一层上实现，如存储过程、触发器，视图等。在该层，服务器会解析并创建相应的内部数据结构（解析树），并完成相应的优化如重写查询、决定表的读取的顺序，选择合适的索引等，最后生成对应的执行操作。如果是select 语句，服务器还会先检查查询缓存（Query Cache），如果能够在其中找到对应的查询，服务器就不必再执行查询解析、优化和执行整个过程，而是直接返回查询缓存中的结果集。如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

### 3. 引擎层(Storage Engines)：
在存储引擎层，存储引擎真正的负责了MySql 中数据的存储和提取，服务器通过API 与存储引擎进行通信。这些接口屏蔽了不同存储引擎之间的差异，使得这些差异对上层的查询过程透明。存储引擎不会去解析SQL（InnoDB是一个例外，它会解析外键定义，因为MySQL服务器本身没有实现该功能），不同存储引擎之间也不会相互通信，而只是简单地响应上层服务器的请求。

### 4.存储层(File System)： 
数据存储层，主要将数据存储在运行于裸设备上的文件系统上，并完成与存储引擎的交互。



## 二. MySQL 主流引擎

特点| MyISAM | InnoDB
-|-|-
事物操作| N | Y
外键关联|N|Y
全文搜索(full text index)| Y | N
锁范围| 表级锁|支持行锁


### 1. MyISAM 仅支持表锁，在海量写入的时候，会导致所有查询处于Locked状态。
### 2. MySQL 5.1 以前版本默认使用 MyISAM, 5.5 版本以上默认 InnoDB。


## 三. MariaDB 的来源
### 1. MySQL被收购之后，面临着被闭源的风险，因此MySQL之父 Widenius离开Sun之后，2009年重新开发代码全部开源免费关系型数据库，推出了MariaDB。
### 2. MariaDB是MySQL的代码级量身定制的替代者，所以大部分功能相似，而且接口一样，相应的版本可以直接替换。
MySQL 版本|MariaDB 版本
-|-
5.5|5.5
5.6|10.0
5.7|10.1


## 四. MySQL 优化步骤
当数据达到千万级时，查询会变得慢下来，一般优化步骤如下
### 1. 优化的sql，使用索引
### 2. 添加缓存，比如 redis, memcached
### 3. 分库，读写分离，主库写数据，从库读，一般上层应用框架都支持读写分离的数据库配置。
### 4. 分区，将数据按照一定规则分布到多个区，避免全区扫描，也可以多个区并发处理；MySQL 自带的分区功能，对上层应用框架是透明的，但是存在多种限制功能，后面会说到。
### 5. 分表，比如根据自增ID每10万条记录分成一个表，应用程序根据ID再去操作具体的表格，这个需要上层应用框架配合修改，成本比较高，除非有足够的理由，否则不太建议。

## 五. MySQL 索引
### 1. 主键和索引
- 主键保证数据行记录的唯一，且不能非 NULL，类比书本的数字页码
- 索引可以不唯一，也可以为空，用于快速检索，类比书本的目录

### 2. 索引主要类型
- 主键索引，主键默认有索引功能，创建主键时自动添加索引。
- 唯一索引，索引值必须唯一，可以为 NULL。
- 普通索引，索引值可以重复，但是重复率太高会影响检索性能。
```sh
# 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL
ALTER TABLE tbl_name ADD PRIMARY KEY (id);
# 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。
ALTER TABLE tbl_name ADD UNIQUE index_name (age);
# 添加普通索引，索引值可出现多次。 
ALTER TABLE tbl_name ADD INDEX index_name (name);
# 创建联合索引
# 因为 MySQL 采用最左前缀方式组合，相当于创建了两个索引: age 和 (age,name)
ALTER TABLE tbl_name ADD INDEX age_name (age,name);
```

### 3. explain 分析索引使用情况
```sql
mysql> explain select * from stock where scode='123456' \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: stock
   partitions: NULL
         type: ref
possible_keys: scode
          key: scode
      key_len: 303
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```
- table, 查询的表格名称
- possible_keys, 可能会使用的索引
- key, 真正使用的索引名称
- rows, 扫描行数，行数越少，代表索引性能越好


### 4. 尽量使用高效索引的方法
- SQL少用like匹配：
```sh
# 是不会使用到索引
like '%test%'
# 这样才会用到索引  
like 'test%' 
```
 
- 不要在字段上进行截取或者运算
```sh
# 这会使索引失效 
select * from post where DATE_FORMAT(time,'%Y%m%d') < 20150225

# 改成这样，效率会高很多 
select * from post where time < '2015-02-25 00:00:00'
```

- 只用一个索引，MySQL查询只使用一个索引，比如
```sh
# where 中的 age 已经使用了索引，后面的order by time是不会使用索引的 
select * from user where age = '25' order by time desc; 
```

- 索引的字段值尽量不要重复，以便提高索引效果。

- 给每个参数添加单引号，以便使用索引。

- 索引列值中不要包含null，因为查询很难优化，而且比空字符串占用空间大。



### 5. 索引缺点
- 降低更新表的速度，如对表进行INSERT、UPDATE和DELETE，不仅要保存数据，还要保存一下索引文件。
- 索引占用表空间，即会占用磁盘空间，所以表数据越多，索引文件也会越多，占用大量磁盘空间。


## 六. MySQL 分区
### 1. 理解分区
- 分区是指同一表中不同行的记录分配到不同的物理文件中，几个分区对应几个物理文件
- 这些物理文件保存在同一台机器上，查看 MySQL 的 datadir 变量可知具体目录，文件后缀是 idb

```sh
mysql> show variables like '%datadir%';
+---------------+-------------------------------------+
| Variable_name | Value                               |
+---------------+-------------------------------------+
| datadir       | D:\wamp\bin\mysql\mysql5.7.14\data\ |
+---------------+-------------------------------------+
1 row in set, 1 warning (0.01 sec)


D:\wamp\bin\mysql\mysql5.7.14\data\my_test_db
2020/04/06  11:13            98,304 test_tb#p#p1.ibd
2020/04/06  11:13            98,304 test_tb#p#p2.ibd
2020/04/06  11:13            98,304 test_tb#p#pmax.ibd
2020/04/06  11:11             8,634 test_tb.frm
```

### 2. 分区方法
分区列必须是唯一索引(主键)的一个组成部分，即列值不能重复，而且必须有索引功能。一般按照时间或者自增ID来分区，方便归档和清理分区数据。
- RANGE : 是实战最常用的一种分区类型，行数据基于属于一个给定的连续区间的列值被放入分区。但是记住，当插入的数据不在一个分区中定义的值的时候，会抛异常。
- LIST : 和RANGE分区很相似，只是分区列的值是离散的，不是连续的。LIST分区使用VALUES IN，因为每个分区的值是离散的，因此只能定义值。
- HASH : 哈希是将数据均匀的分布到预先定义的各个分区中，保证每个分区的数量大致相同。
- KEY : 和HASH分区相似，不同之处在于HASH分区使用用户定义的函数进行分区，KEY分区使用数据库提供的函数进行分区。


### 3. 测试分区
- 创建分区表，以自增ID为分区列，通过 range 方式分区
```sh
CREATE TABLE `test_tb` (
  `id` int(11) NOT NULL,
  `insert_at` datetime NOT NULL,
  `order_id` char(200) default '' not null,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
PARTITION BY RANGE (id)(
	PARTITION p1 VALUES LESS THAN (2) ENGINE = InnoDB,
	PARTITION p2 VALUES LESS THAN (4) ENGINE = InnoDB,
	PARTITION pmax VALUES LESS THAN maxvalue ENGINE = InnoDB
);
```

- 插入数据
```sh
INSERT INTO `test_tb` (`id`,`insert_at`,`order_id`) VALUES (1, now(), '100');
INSERT INTO `test_tb` (`id`,`insert_at`,`order_id`) VALUES (2, now(), '200');
INSERT INTO `test_tb` (`id`,`insert_at`,`order_id`) VALUES (3, now(), '300');
INSERT INTO `test_tb` (`id`,`insert_at`,`order_id`) VALUES (4, now(), '400');
INSERT INTO `test_tb` (`id`,`insert_at`,`order_id`) VALUES (5, now(), '500');
INSERT INTO `test_tb` (`id`,`insert_at`,`order_id`) VALUES (6, now(), '600');
```

- 查看每个分区的数据情况
```sh
mysql> select PARTITION_NAME ,TABLE_ROWS  from information_schema.partitions where table_schema="my_test_db" and table_name="test_tb";
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p1             |          1 |
| p2             |          2 |
| pmax           |          3 |
+----------------+------------+
3 rows in set (0.00 sec)
```

- 分析 SQL 在哪个区域执行，可通过 partitions 参数知道
```sh

mysql> explain partitions select * from test_tb where id = 2 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: test_tb
   partitions: p2
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 2 warnings (0.00 sec)

mysql> explain partitions select * from test_tb where id = 4 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: test_tb
   partitions: pmax
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 2 warnings (0.00 sec)


```



### 4. 分区的限制性功能
- 分区的 innodb 表不支持外键，这个要非常小心。
- 分区不支持全文搜索，不管是 InnoDB 还是 MyISAM 引擎。

