---
title: mysql 隔离级别
date: 2020-05-08 07:20:28
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->


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


### 解决幻读的方法
1. 采用串行化的隔离级别，虽然能避免幻读，但是性能将会下降

2. 快照度时，采用多版本并发 MVCC 机制

InnoDB的MVCC，是通过在每行记录后面保存两个隐藏的列来实现的。
这两个列，一个保存了行的创建时间，一个保存行的过期时间（或删除时间）。当然存储的并不是实际的时间值，而是系统版本号（systemversionnumber）。
每开始一个新的事务，系统版本号都会自动递增。事务开始时刻的系统版本号会作为事务的版本号，用来和查询到的每行记录的版本号进行比较。

MVCC快照读需满足条件：
```sql
select * from tb where ?
```
- InnoDB只查找版本早于当前事务版本的数据行（也就是，行的系统版本号小于或等于事务的系统版本号），这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。
- 行的删除版本要么未定义，要么大于当前事务版本号。这可以确保事务读取到的行，在事务开始之前未被删除。

MVCC解决了基于快照读下的幻读，事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。并不会读到其他事务的写操作！！！


3. 当前读

```sql
select * from tb where ? lock in share mode;
select * from tb where ? for update;
```

for update：IX锁(意向排它锁)，即在符合条件的rows上都加了排它锁
lock in share mode：是IS锁(意向共享锁)，即在符合条件的rows上都加了共享锁
排它锁：X锁、写锁，事务A对一个资源加了X锁后只有A本身能对该资源进行读和写操作，其他事务对该资源的读和写操作都将被阻塞，直到A释放锁为止
共享锁：S锁、读锁，事务A锁定的数据其他事务可以共享读该资源，但不能写，直到事务A释放


```sql
start transaction;
select * from tb where id>100 for update;
update tb set product_num=product_num-1 where id>100;
``` 


Next-KeyLock 是 GapLock（间隙锁）和 RecordLock（行锁）的结合版，都属于Innodb的锁机制

```sql
select * from tb where id>100
```
主键索引id会给id=100的记录加上record行锁，索引大于id会加上gap锁，锁住id(100,+无穷大）这个范围其他事务对id>100范围的记录读和写操作都将被阻塞，插入id=1000的记录时候会命中索引上加的锁会报出事务异常；
Next-KeyLock会确定一段范围，然后对这个范围加锁，保证A在where的条件下读到的数据是一致的，因为在where这个范围其他事务根本插不了也删不了数据，都被Next-KeyLock锁堵在一边阻塞掉了。


### MySQL 事物默认自动提交，通过以下命令查看和修改，修改全局 global 级别会对所有登录用户生效，也可以仅对当前 session 修改。

```sql
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
```sql

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

```sql

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


