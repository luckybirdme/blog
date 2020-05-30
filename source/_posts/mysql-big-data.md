---
title: 一亿数据的表字段修改
date: 2020-05-19 08:29:13
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->



### 一. MySQL DDL 的问题现状

##### 1. 问题
MySQL 对表进行ddl时，会锁表，当表比较小，比如小于1w条记录时，操作时间较短，对前端影响较小，当时遇到千万乃至上亿级级别的表，就会影响前端应用对表的写操作。

##### 2. 过程：
目前InnoDB引擎是通过以下步骤来进行 DDL 的：
- 按照原始表（original_table）的表结构和DDL语句，新建一个不可见的临时表（tmp_table）

- 在原表上加 write lock，阻塞所有更新操作 insert、delete、update等，但是可以读

- 执行数据复制 insert into tmp_table select * from original_table

- 交换表名，rename original_table 和 tmp_table，最后 drop original_table

- 释放 write lock。

可以看见在InnoDB执行DDL的时候，原表是只能读不能写的。为此 perconal 推出一个工具 pt-online-schema-change ，其特点是修改过程中不会造成读写阻塞。



### 二. pt-online-schema-change 介绍


##### 1. 特点
pt-online-schema-change 模仿MySQL内部的改表方式进行改表，但整个改表过程是通过对原始表的拷贝来完成的，即在改表过程中原始表不会被锁定，并不影响对该表的读写操作。

##### 2. 过程
- 创建与原始表相同的不包含数据的新表
- 按照需求进行表结构的修改
- 将原始表中的数据按 chunk 大小逐步拷贝到新表中
- 在原始表添加一个触发器，将原始表新增的数据也拷贝过来
- 当拷贝完成后，会自动同时修改原始表和新表的名字，并默认将原始表删除


##### 3. 实例

- 创建两个和你要执行 alter 操作的表结构一样的空表。如图：

```sql
/*
说明：
t_ad_req_log 就是原表

_t_ad_req_log_old 是旧表，这个表是用来当你执行失败的时候，还原回来的原表结构；

_t_ad_req_log_new 是新表，这个表就是这次要修改的表。
*/
```

- 执行表结构修改，然后从原表中的数据到copy到表结构修改后的表，即_t_ad_req_log_new

- 在原表上创建触发器将 copy 数据的过程中，在原表的更新操作也更新到新表。注意:如果表中已经定义了触发器这个工具就不能工作了。

- copy 完成以后，用 rename table 新表代替原表，默认删除原表。

 

- 具体修命令如下：

```sql
/usr/local/bin/pt-online-schema-change --user='test' --password='123456' --host=127.0.0.1 --port=3306 --charset=utf8 --nodrop-old-table --alter="modify media_code varchar(64) DEFAULT NULL" D=db_name,t=tb_name --exec

```