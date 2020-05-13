---
title: mysql 索引
date: 2020-05-08 07:26:22
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->


### 1. 主键和索引
- 主键保证数据行记录的唯一，且不能非 NULL，类比书本的数字页码
- 索引可以不唯一，也可以为空，用于快速检索，类比书本的目录

### 2. 索引主要类型
- 主键索引，主键默认有索引功能，创建主键时自动添加索引。
- 唯一索引，索引值必须唯一，可以为 NULL。
- 普通索引，索引值可以重复，但是重复率太高会影响检索性能。
- 组合索引，两个以上字段作为索引，采用最左前缀方式使用索引。
- 全文索引，MyISAM 默认支持，MySQL 5.7 引擎 InnoDB 开始支持。


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
# 创建全文搜索
ALTER TABLE tbl_name ADD FULLTEXT INDEX input_content(input_content) with parser ngram;

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
