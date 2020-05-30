---
title: MySQL binlog
date: 2020-05-24 16:39:04
tags: MySQL
---


> 本文基于 MySQL 5.7 (10.1.9-MariaDB)

<!-- more -->

###  MySQL 主要日志
MySQL Server 有四种类型的日志：
##### 1. Error Log
错误日志，记录 mysqld 的一些错误。

##### 2. General Query Log
一般查询日志，记录 mysqld 正在做的事情，比如客户端的连接和断开、来自客户端每条 Sql Statement 记录信息；如果你想准确知道客户端到底传了什么SQL给服务端，这个日志就非常管用了，不过它非常影响性能。

##### 3. Slow Query Log
是慢查询日志，记录一些查询比较慢的 SQL 语句——这种日志非常常用，主要是给开发者调优用的。默认超时 10s 会记录，可通过 long_query_time 设置。

##### 4. Binary Log
就是 Binlog 了，binlog 是一个二进制格式的文件，用于记录用户对数据库更新的SQL语句信息。binlog 包含了一些事件，这些事件描述了数据库的改动，如建表、数据改动等。Binlog 日志的两个最重要的使用场景：

- MySQL主从复制：MySQL 在Master端开启binlog，Master把它的二进制日志传递给slaves来达到master-slave数据一致的目的
- 数据恢复：在误操作数据后，通过使用 mysqlbinlog 工具来使恢复数据。



### binlog 开启

##### 1. log_bin = ON, 表示开启

##### 2. log_bin_basename, 记录数据变化的 binlog 的前缀名称

##### 3. log_bin_index, 记录索引变化的 binlog 文件

```sql
mysql> show variables like "%log_bin%";
+---------------------------------+---------------------------------------------
| Variable_name                   | Value
+---------------------------------+---------------------------------------------
| log_bin                         | ON
| log_bin_basename                | D:\wamp\bin\mysql\mysql5.7.14\data\mysql-bin
| log_bin_index                   | D:\wamp\bin\mysql\mysql5.7.14\data\mysql-bin.index 
+---------------------------------+---------------------------------------------
```

### binlog 写时机

##### 1. 通过参数 sync_binlog 控制

- 如果设置为0，则表示MySQL不控制binlog的刷新，由文件系统去控制它缓存的刷新；
- 如果设置为不为0的值，则表示每 sync_binlog 次事务，MySQL调用文件系统的刷新操作刷新binlog到磁盘中。
- 设为1是最安全的，在系统故障时最多丢失一个事务的更新，但是会对性能有所影响。

```sql
mysql> show variables like "%sync_binlog%";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| sync_binlog   | 1     |
+---------------+-------+
1 row in set, 1 warning (0.01 sec)

```

### binlog 文件

##### 1. 通过参数 max_binlog_size 设置，默认是 1 G

```sql
mysql> show variables like "%max_binlog_size%";
+--------------------------+------------+
| Variable_name            | Value      |
+--------------------------+------------+
| max_binlog_size          | 1073741824 |
+--------------------------+------------+
```

##### 2. 当前 binlog 文件超过设定值时，将会以自增ID创建一个新的 binlog 文件

```sql
mysql> show binary logs
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |  10343266 |
| mysql-bin.000002 |  10485660 |
| mysql-bin.000003 |     53177 |
+------------------+-----------+

```

##### 3. 也可以通过命名产生一个新编号的binlog日志文件

```sql
mysql> flush logs;
Query OK, 0 rows affected (0.04 sec)

```


### binlog 内容格式


##### 1. binlog 内容是二进制，通过命名查看当前 binlog 文件以及具体内容，包括当前的执行位置 position

```sql
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
mysql> show binlog events in 'mysql-bin.000001' limit 10 \G
*************************** 1. row ***************************
   Log_name: mysql-bin.000001
        Pos: 4
 Event_type: Format_desc
  Server_id: 1
End_log_pos: 123
       Info: Server ver: 5.7.14-log, Binlog ver: 4
*************************** 2. row ***************************
   Log_name: mysql-bin.000001
        Pos: 123
 Event_type: Previous_gtids
  Server_id: 1
End_log_pos: 154
       Info:
```

##### 2. 通过 mysqlbinlog 命令转换成，即可看到具体的 SQL， at 后面的数字就是 position

```sh
# ROW 模式下的 SQL 是  base64 编码的，所以需要转换
[root@VM_133_231_centos ~]# mysqlbinlog  --base64-output=decode-rows -v /data/mysql-bin.000002 > /data/2.txt
[root@VM_133_231_centos ~]# cat /data/2.txt
# at 867
#200524 17:51:19 server id 1  end_log_pos 929 CRC32 0x1a43bd05  Table_map: `test_file_db`.`tt_ff_tb` mapped to number 108
# at 929
#200524 17:51:19 server id 1  end_log_pos 974 CRC32 0x439dead8  Write_rows: table id 108 flags: STMT_END_F
### INSERT INTO `test_file_db`.`tt_ff_tb`
### SET
###   @1=2
###   @2='two'
# at 974
#200524 17:51:19 server id 1  end_log_pos 1005 CRC32 0xc10ed433         Xid = 31

COMMIT/*!*/;
# at 1005
#200524 17:51:41 server id 1  end_log_pos 1070 CRC32 0xc519316b         Anonymous_GTID  last_committed=3        sequence_number=4
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1070
#200524 17:51:41 server id 1  end_log_pos 1150 CRC32 0x9cf3754c         Querythread_id=2     exec_time=0     error_code=0
SET TIMESTAMP=1590313901/*!*/;
BEGIN
/*!*/;
# at 1150
#200524 17:51:41 server id 1  end_log_pos 1212 CRC32 0x9e3239da         Table_map: `test_file_db`.`tt_ff_tb` mapped to number 108
# at 1212
#200524 17:51:41 server id 1  end_log_pos 1258 CRC32 0xd9bb3b6b         Delete_rows: table id 108 flags: STMT_END_F
### DELETE FROM `test_file_db`.`tt_ff_tb`
### WHERE
###   @1=20
###   @2='four'
# at 1258
#200524 17:51:41 server id 1  end_log_pos 1289 CRC32 0xd603b818         Xid = 32

COMMIT/*!*/;

```

##### 3. binlog 内容格式通过参数 binlog_format 设置，主要有三种模式

```sql
mysql> show variables like "%binlog_format%";
+---------------------------+-------------------+
| Variable_name             | Value             |
+---------------------------+-------------------+
| binlog_format             | ROW               |
```

- Statement, 基于SQL语句的复制（statement-based replication, SBR）
每一条会修改数据的 SQL 都会记录在binlog中。
如果 SQL 包含了特殊功能函数，比如 now(), uuid(), 则无法却确保函数执行时的数值时相同的，具体例子: update my_tb set create_time=now() where id=3, 不同时间执行 now() 函数 将会有不同的值。


- Row, 基于行的复制（row-based replication, RBR）
它不记录修改数据的 SQL 语句，仅保存行数据被修改成的最终状态。
row的日志内容会非常清楚的记录下每一行数据修改后的细节，类似 Statement 模式下的问题，也是 MySQL Server 默认的模式。此模式的缺点是所有执行的SQL都会记录，包括查询语句 select，所以日志文件增长会比较快。


- Mixed, 混合模式复制（mixed-based replication, MBR）
Statement 和 Row 混合模式，这里不细说


```sql

mysql> show binlog events in 'mysql-bin.000001' limit 10 \G
*************************** 4. row ***************************
   Log_name: mysql-bin.000001
        Pos: 219
 Event_type: Query
  Server_id: 1
End_log_pos: 299
       Info: BEGIN
*************************** 5. row ***************************
   Log_name: mysql-bin.000001
        Pos: 299
 Event_type: Table_map
  Server_id: 1
End_log_pos: 361
       Info: table_id: 108 (test_file_db.tt_ff_tb)
*************************** 6. row ***************************
   Log_name: mysql-bin.000001
        Pos: 361
 Event_type: Write_rows
  Server_id: 1
End_log_pos: 407
       Info: table_id: 108 flags: STMT_END_F

·····

*************************** 26. row ***************************
   Log_name: mysql-bin.000001
        Pos: 1496
 Event_type: Delete_rows
  Server_id: 1
End_log_pos: 1542
       Info: table_id: 108 flags: STMT_END_F
*************************** 27. row ***************************
   Log_name: mysql-bin.000001
        Pos: 1542
 Event_type: Xid
  Server_id: 1
End_log_pos: 1573
       Info: COMMIT /* xid=33 */
```



### binlog 恢复数据

##### 1. 原理
因为 binlog 记录了所有 SQL 操作记录，当误操作删除了数据，确定误操作前的 binlog 记录位置，然后将之前的 binlog 日志记录的 SQL 重新一次，即可恢复数据。


##### 2. 恢复方式主要有两种
- 将所有 binlog 文件重头到尾全部执行一次，虽然能恢复所有数据，但是要把之前所有执行过的 SQL 都得重新一次， 效率相对较低。
- 先将定期备份的数据全量恢复，比如凌晨 mysqldump 出来的数据，先执行一次性恢复，然后再将凌晨到现在的 binlog 日志记录的 SQL 重新执行，那么效率会高好多，这也是常用的恢复方式。

##### 3. 常用的恢复方式


- 从定期备份文件执行一次性恢复

```sh
[root@VM_133_231_centos ~]# mysql -uroot -p -v < my_tb_bak_2020-05-25.sql 
```

- 找到定期备份之后的 binlog 日志，开始位置就是凌晨日志记录的位置，误操作的停止位置 End_log_pos=1718

```sql

# at 1573
#200524 20:18:46 server id 1  end_log_pos 1638 CRC32 0x6b15c4cb         Anonymous_GTID  last_committed=5        sequence_number=6
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1638
#200524 20:18:46 server id 1  end_log_pos 1718 CRC32 0x2cb80669         Querythread_id=2     exec_time=0     error_code=0
SET TIMESTAMP=1590322726/*!*/;
BEGIN
/*!*/;
# at 1718
#200524 20:18:46 server id 1  end_log_pos 1780 CRC32 0x00b841ab         Table_map: `test_file_db`.`tt_ff_tb` mapped to number 108
# at 1780
#200524 20:18:46 server id 1  end_log_pos 1835 CRC32 0x9ecfcbe7         Delete_rows: table id 108 flags: STMT_END_F
### DELETE FROM `test_file_db`.`tt_ff_tb`
### WHERE
###   @1=1
###   @2='one'
### DELETE FROM `test_file_db`.`tt_ff_tb`
### WHERE
###   @1=2
###   @2='two'
# at 1835
#200524 20:18:46 server id 1  end_log_pos 1866 CRC32 0x7154e1ec         Xid = 44

COMMIT/*!*/;

```


- 执行 binlog 指定区间的日志，恢复剩余的数据

```sh
mysqlbinlog  /data/mysql-bin.000002 --start-pos=759 --stop-pos=1718 | mysql -uroot -p123456

```


### binlog 主从同步

##### 1. 原理
- Master将数据改变记录到二进制日志 binlog 中
- Slave上面的IO进程连接上Master，并请求从指定日志文件的指定位置之后的日志内容
- Master接收到来自Slave的IO进程的请求后，负责复制的IO进程会根据请求信息读取日志指定位置之后的日志信息，返回给Slave的IO进程。
- Slave的IO进程接收到信息后，将接收到的日志内容依次添加到Slave端的relay-log文件的最末端，并将读取到的Master端的 binlog 的文件名和位置记录到master-info文件中，以便在下一次读取的时候能够清楚的告诉Master从某个binlog的哪个位置开始往后的日志内容。
- Slave的Sql进程检测到relay-log中新增加了内容后，会马上解析relay-log的内容成为在Master端真实执行时候的那些可执行的内容，并在自身执行。


##### 2. 主从同步：主库 master 配置

- 配置my.cnf

```sh
# 唯一标志ID
server-id=1
# master 数据操作记录的二进制日志，提供给 slave 读取 
log-bin=/var/lib/mysql/log/bin
# master 指定同步的数据库
binlog_do_db=test

```

- 重启 mysql

```sh
[mysql@VLR-123-177-206 ~]#  service mysql restart
```

 

- 在 master 上创建 slave 用户，提供给 slave 登录来同步数据

```sql
mysql>create user slave_server; 
/*
只需 REPLICATION SLAVE 权限，除此之外没有必要添加不必要的权限，密码为 slave_passwd。
这里 % 是通配符，表示10.123.189.0-10.123.189.255的从服务器都能以 slave_server 用户登陆主服务器。
*/
mysql>GRANT REPLICATION SLAVE ON *.* TO 'slave_server'@'10.123.189.%' IDENTIFIED BY 'slave_passwd';
 
```

 

- 导出 master 中 test 数据库的原始数据，以便快速迁移数据

```sql

/*
暂时把 test 表锁住，保证导出的数据一致性
*/
mysql>use test;
mysql>FLUSH TABLES WITH READ LOCK;
/*
执行数据导出
*/
[mysql@VLR-123-177-206 ~]# mysqldump -u root -p --opt test > ~/test.sql
 
```
 

- 查看 master 当前数据操作记录的文件 binlog 和起始点 position，后面配置 slave 会用到这些信息

```sql

mysql> show master status;
+------------+----------+--------------+------------------+
| File       | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------+----------+--------------+------------------+
| bin.000001 |      107 | test         |                  |
+------------+----------+--------------+------------------+
1 row in set (0.00 sec)

/*
这个时候可以释放 test 表格了
*/
mysql>use test;
mysql>UNLOCK TABLES;
```
 


##### 3. 主从复制：主库 slave 配置


- 配置my.cnf，并重启 slave 数据库
```sh
# 唯一标志ID
server-id=2
# slave 获取 master 数据的记录
relay-log=/var/lib/mysql/relay/log

# 记得重启 mysql
# [mysql@VLR-123-189-150 ~]#  service mysql restart

```

- 导入 test 数据库的原始数据到 slave 数据库

```sh
[mysql@VLR-123-189-150 ~]#   mysql -u root -p test < ~/test.sql

```

- slave 指定 master 的同步信息，包括 master 的 host,port,user,password, 以及 binlog 文件，以及同步起始点 position

```sql
mysql> CHANGE MASTER TO MASTER_HOST='10.123.177.206',MASTER_USER='slave_server', MASTER_PASSWORD='slave_passwd', MASTER_LOG_POS=3306,master_log_file='bin.000001', master_log_pos=107;
Query OK, 0 rows affected (0.05 sec)

```
 

- 开启 slave 同步，并查看状态，如果有 Waiting for master to send event 表示成功了

```sql
/* 开启 slave 同步 */
mysql> START SLAVE;
Query OK, 0 rows affected (0.01 sec)
/* 查看 slave 状态 */
mysql> show slave status;
+----------------------------------+----------------+--------------+-------------+---------------+-----------------+---------------------+
| Slave_IO_State                   | Master_Host    | Master_User  | Master_Port | Connect_Retry | Master_Log_File | Read_Master_Log_Pos | 
+----------------------------------+----------------+--------------+-------------+---------------+-----------------+---------------------+
| Waiting for master to send event | 10.123.177.206 | slave_server |        3306 |            60 | bin.000001      |                 107 |
+----------------------------------+----------------+--------------+-------------+---------------+-----------------+---------------------+
1 row in set (0.00 sec)

```