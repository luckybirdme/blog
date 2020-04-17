---
title: Linux find 命令
date: 2020-04-14 07:35:20
tags: Linux
---

> find

<!-- more -->

- find, 只在当前目录层查找
```sh
[root@~ ]# find . -maxdepth 1 
.
./log-2019-02-09.php
./log-2020-02-01.php
```
- find, 只查找文件
```sh
[root@~ ]# find . -maxdepth 1 -type f
./log-2019-02-09.php
./log-2020-02-01.php
```

- find, 查找指定文件名
```sh
[root@~ ]# find . -maxdepth 1 -name "*.txt"
./one.txt
./two.txt
```

- find, 查找指定用户的文件
```sh
[root@~ ]# find . -maxdepth 1 -type f -user ftp | xargs ls -l
-rw-rw-r-- 1 ftp ftp 2.7K Mar 31 20:51 ./ftp.log
```

- find, 查找大于 1M 的文件
```sh
[root@~ ]# find . -maxdepth 1 -type f -size +1M | xargs ls -lhS | head -n 10
-rw-r--r-- 1 root   root    16M Mar  4 14:24 ./sendUnConfirm.log
-rw-r--r-- 1 root   root    12M Dec 31 14:14 ./reservationSendMsg.log
-rw-rw-rw- 1 root   root   9.7M Oct 12  2019 ./django.log.2019-10-12_19
```

- find, 对查到到的结果执行 shell 命令, [-exec] 是命令开头, [{}] 是查到的结果, [\;] 是命令结尾
```sh
[root@~ ]# find . -name "country_data.txt" -exec grep "CN" {} \;
2019-09-04      24506500        24442850        81.37   22.44   CN      中国    1567611699
```

- find, 四天前当天的文件
```sh
[root@~ ]# date
Mon Apr 13 20:13:34 CST 2020
[root@~ ]# find . -maxdepth 1 -type f -mtime 3
./log-2020-04-09.php
```

- find, 四天以前的文件，不包括当天的文件
```sh
[root@~ ]# find . -maxdepth 1 -type f -mtime +3
./log-2020-04-08.php
./log-2020-04-07.php
./log-2020-04-06.php
```

- find, 四天内的文件，包括当天的文件
```sh
[root@~ ]# find . -maxdepth 1 -type f -mtime -3
./log-2020-04-10.php
./log-2020-04-11.php
./log-2020-04-12.php
./log-2020-04-13.php
```

- find, 四天内的文件，包括当天，两天前的文件，不包括当天
```sh
[root@~ ]# date
Mon Apr 13 20:19:38 CST 2020
[root@~ ]# find . -maxdepth 1 -type f -mtime -3 -mtime +1
./log-2020-04-10.php
```

![](http://qiniucdn.luckybird.me/blog/img/2020/find_mtime.png)
