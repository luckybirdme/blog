---
title: MySQL 知识点
date: 2020-04-04 17:41:09
tags: MySQL
---

> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->


### union 和 union all
- union 去掉重复数据
- union all 包括重复数据



### left join 保留左边的所有数据，right join 保留右边
```sql
/*
有这么两个表
user 表：

id  name
1   张三
2   李四
3   王五
4   赵六

apple 表：

id  user  number
1   1     5
2   3     3
3   1     8
4   4     6
5   3     2
6   4     2

apple 表的 user 字段跟 user 表的 id 对应，一条 SQL 语句查出每个人都有多少苹果
*/

/*
不用 left join
*/
SELECT user.name, SUM(apple.number) FROM user, apple WHERE user.id = apple.user GROUP BY user.id

/*
正确答案应该是这样
*/
SELECT user.name, SUM(apple.number) FROM user LEFT JOIN apple ON user.id = apple.user GROUP BY id

```

### MySQL 临时表用于保存临时数据，show tables 下无法查看，而且只在当前连接操作，当关闭连接时，Mysql会自动删除表并释放所有空间。

```sql
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
```sql
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
```sql

mysql> SELECT * FROM stock INTO OUTFILE 'D:/wamp/tmp/stock.txt';
Query OK, 898 rows affected (0.01 sec)

mysql> create table stock_back like stock;
Query OK, 0 rows affected (0.01 sec)

mysql> load data infile  'D:/wamp/tmp/stock.txt' into table stock_back;
Query OK, 898 rows affected (0.02 sec)
Records: 898  Deleted: 0  Skipped: 0  Warnings: 0
```



