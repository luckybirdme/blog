---
title: mysql 锁机制
date: 2020-05-08 07:29:13
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB), InnoDB 引擎

<!-- more -->

## 锁分类

![](/img/2020/mysql_lock.png)

#### 1. 乐观锁与悲观锁是两种并发控制的思想，可用于解决丢失更新问题：
- 悲观锁, 就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。
- 乐观锁，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制，乐观锁适用于多读的应用类型，这样可以提高吞吐量


#### 2. InnoDB支持多种锁粒度，由执行计划决定使用哪种锁，默认是使用行锁，也支持手动设置锁粒度。
- 表锁，会锁住整个表，锁冲突概率高，并发度较低；但可以一次性获取所需要的全部锁，要不成功要不失败，避免了死锁；
- 行锁，以行记录加锁，锁冲突概率较低，并发度较高；如果事物要加锁多个表，无法一次加锁，多个事物产生了锁依赖，容易出现死锁。


#### 3. 共享锁与排他锁是 InnoDB 引擎实现的两种标准的行锁，为了允许行锁和表锁共存，实现多粒度锁机制，InnoDB还有两种内部使用的意向锁。

- 共享锁（S）：允许获取共享锁的事物读取行数据，但是不允许其他事物获取行的排他锁。
- 排他锁（X）：允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。
- 意向共享锁（IS）：事务打算给数据行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。
- 意向排他锁（IX）：事务打算给数据行加排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁。

如果一个事务请求的锁模式与当前的锁兼容，InnoDB就请求的锁授予该事务；反之，如果两者两者不兼容，该事务就要等待锁释放。

锁模式|	X	|IX	|S	|IS
-|-|-|-|-
X	|冲突	|冲突	|冲突	|冲突
IX	|冲突	|兼容	|冲突	|兼容
S	|冲突	|冲突	|兼容	|兼容
IS	|冲突	|兼容	|兼容	|兼容


## 使用何种锁

#### 1. 表锁使用，最常用是通过 LOCAK TABLES

```sql
SET AUTOCOMMIT=0;
LOCAK TABLES t1 WRITE, t2 READ, ...;
[do something with tables t1 and here];
COMMIT;
UNLOCK TABLES;

```

#### 2. 行锁使用，InnoDB 行锁是通过索引来实现的，只有通过索引条件检索数据才会使用行锁，也可以手动设置。

共享锁（Ｓ）

```sql
SELECT * FROM table_name WHERE 1=1 LOCK IN SHARE MODE

``` 

排他锁（X）

``` mysql
SELECT * FROM table_name WHERE 1=1 FOR UPDATE
```

#### 3. 使用何种锁
- 意向锁是 InnoDB 引擎自动添加的，不需用户干预。
- 只有通过索引条件检索数据，InnoDB才会使用行级锁，否则，InnoDB将使用表锁！
- 即使在条件中使用了索引，但是是否使用索引来检索数据是由 MySQL 通过判断不同执行计划的代价决定的，如果MySQL认为全表扫描效率更高，比如很小的表，也不会使用索引，此时InnoDB将使用表锁，而不是行锁。
- 行锁是针对索引加的锁，不是针对记录加的锁，虽然是访问不同的行，但是若是相同的索引，也会出现锁冲突的。
- 对于UPDATE、DELETE和INSERT语句，InnoDB 会自动给涉及及数据集加排他锁（Ｘ）；对于普通SELECT语句，InnoDB不会加任何锁；



## 如何解决死锁

#### 1. 死锁产生的情况
- ＭyISAM 表锁是 deadlock free的，这是因为ＭyISAM总是一次性获得所需的全部锁，要么全部满足，要么等待，因此不会出现死锁。但是在InnoDB中，除单个SQL组成的事务外，锁是逐步获得的，这就决定了InnoDB发生死锁是可能的。
- 发生死锁后，InnoDB一般都能自动检测到，并使一个事务释放锁并退回，另一个事务获得锁，继续完成事务。但在涉及外部锁，或涉及锁的情况下，InnoDB并不能完全自动检测到死锁，这需要通过设置锁等待超时参数innodb_lock_wait_timeout来解决。
-  通常来说，死锁都是应用设计的问题，通过调整业务流程、数据库对象设计、事务大小、以及访问数据库的SQL语句，绝大部分都可以避免。


#### 2. 避免死锁的方法

- 若不同程序并发存取多个表，应约定以相同的顺序来访问表，这样可以降低死锁的机会。
- 在程序中以批量的方式处理数据时，如果事先对数据进行排序，保证每个线程按固定顺序处理记录，也可以大大降低死锁可能。
- 在事务中，如果要更新记录，应该一次性申请足够级别的锁，比如排它锁。不要先申请共享锁，再申请排他锁，这样可能会造成死锁。
- 一个锁定记录集的事务，其操作结果集应尽量简短，以免一次占用太多资源，与其他事务处理的记录冲突。
- 如果使用 insert … select 语句备份表格且数据量较大，可考虑使用 select into outfile，然后load data infile 代替，这样不仅快，而且不会要求锁定太长时间。



#### 3. 解决死锁的方法
- show innodb status 检查引擎状态 ,可以看到哪些语句产生死锁
- 执行 show processlist 找到死锁线程号，然后Kill pid
- 查看 information_schema 架构下的innodb_locks、innodb_trx、innodb_lock_waits等表
