---
title: mysql 分区
date: 2020-05-08 07:26:38
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->


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
- 因为分区的 innodb 表把不同数据保存不同的物理文件，所以不支持外键。
- 不同的物理文件，无法进行全文搜索，不管是 InnoDB 还是 MyISAM 引擎。
