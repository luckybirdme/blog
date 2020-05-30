---
title: MySQL 表空间文件结构
date: 2020-05-24 14:54:01
tags: MySQL
---


> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->


### 经典镇楼图

![](/img/2020/mysql_file_detail_0.png)


### 文件结构

InnoDB 引擎的表空间采取 B+树的结构

##### 1. leaf node, 叶子节点, 用于保存数据，是行记录最终落地保存的地方，叶子节点之间采用双向链表，方便查找指定范围的关联数据。

##### 2. non-leaf node, 非叶子节点, 用于检索的索引段，不保存数据，所以能够容纳更多的 key，降低B+树的高度，提高检索效率。


![](/img/2020/mysql_file_detail_1.png)

### 文件内容
表空间由段（segment），簇/区（extend），页（page）组成。它们是逻辑概念词，便于进行资源管理，比如数据变多时，申请多个一个区(extend)，即加多 64 页(page)来保存数据。

##### 1. 段
- 表空间最顶部的分层，主要包括数据段、索引段、回滚段等。
- InnoDB 引擎的表空间是索引和数据同在一颗B+树上(聚簇索引，主键索引)。那么数据段即为B+树的叶子节点 leaf node segment ，索引段即为B+树的非叶子节点 non-leaf node segment

##### 2. 簇/区
- 簇是构成段的基本元素，一个段由若干个簇构成，每一个段至少会有一个簇，在创建一个段时会创建一个默认的簇。
- 簇是物理上连续分配的空间，一个簇已经不足以放下更多的数据，此时需要从段中分配一个新的簇来存放新的数据。
- 一个段所管理的空间大小是无限的，可以一直扩展下去，但是扩展的最小单位就是簇。
- 簇由 64 个连续的页组成的，默认每个页大小为16KB，每个簇的大小为 16KB\*64=1MB。


##### 3. 页
- 页是簇的组成单位，是磁盘空间分配的最小单位，默认每个页大小 16KB，可通过 innodb_page_size 修改。
- 页面号都是从小到大连续的，在物理上都是连续的。
- 在向表中插入数据时，如果一个页面已经被写完，系统会从当前簇中分配一个新的空闲页面处理使用。
- 如果当前簇中的64个页面都被分配完，系统会从当前段中分配一个新的簇，然后再从这个簇中分配一个新的页面来使用。

常见页面类型：
数据页（B-tree Node）。
Undo页（Undo Log Page）。
系统页（System Page）。
事务数据页（Transaction system Page）。
插入缓冲位图页（Insert Buffer Bitmap）。
插入缓冲空闲列表页（Insert Buffer Free List）。
未压缩的二进制大对象页（Uncompressed BLOB Page）。
压缩的二进制大对象页（Compressed BLOB Page）


```sql
mysql> show variables like "%innodb_page_size%" ;
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| innodb_page_size | 16384 |
+------------------+-------+
1 row in set, 1 warning (0.01 sec)

```


![](/img/2020/mysql_file_detail_2.png)




