---
title: Linux 常用命令
date: 2020-04-13 08:06:38
tags: Linux
---

> cmd

<!-- more -->

- umask, 设置新建目录或文件时去掉的权限, 创建文件总权限是 666, 即不包括执行权限[x], 创建目录总权限是 777 
```sh
# 显示去掉的权限, 后三个数字对于用户、用户组、其他用户的去掉权限, 即去掉[w]权限
[root@VM_133_231_centos ~]# umask
0022
# 创建文件
[root@VM_133_231_centos ~]# touch test_new_file.txt
# 显示文件权限, 默认没有执行权限[x], 根据 umask 设置去掉权限, 666-022=644
[root@VM_133_231_centos ~]# ls -l test_new_file.txt 
-rw-r--r-- 1 root root 0 Apr 13 08:11 test_new_file.txt
# 创建目录
[root@VM_133_231_centos ~]# mkdir test_new_dir
# 显示目录权限, 总权限是 777, 根据 umask 设置去掉权限, 777-022=755
[root@VM_133_231_centos ~]# ls -ld test_new_dir 
drwxr-xr-x 2 root root 4096 Apr 13 08:13 test_new_dir
``` 

- file, 查看文件类型
```sh
[root@VM_133_231_centos ~]# file .bashrc 
.bashrc: ASCII text
[root@VM_133_231_centos ~]# file popen.py          
popen.py: Python script, ASCII text executable
[root@VM_133_231_centos ~]# file /bin/ls
/bin/ls: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=3d705971a4c4544545cb78fd890d27bf792af6d4, stripped
```

- which, 搜索命令所在路径及别名
```sh
[root@VM_133_231_centos ~]# which ls
alias ls='ls --color=auto'
	/usr/bin/ls
[root@VM_133_231_centos ~]# which ifconfig
/usr/sbin/ifconfig

```

- whereis, 搜索命令所在的路径以及帮助文档所在的位置
```sh
[root@VM_133_231_centos ~]# whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
# 搜索的目录
[root@VM_133_231_centos ~]# whereis -l
bin: /usr/bin
bin: /usr/sbin
bin: /usr/lib
···
```

- tar, 压缩排除目录
```sh
[root@VM_133_231_centos test_tar]# ls -rhtl
total 12K
drwxr-xr-x 2 root root 4.0K Apr 14 08:46 one_dir
drwxr-xr-x 2 root root 4.0K Apr 14 08:47 two_dir
-rw-r--r-- 1 root root    6 Apr 14 08:47 three.txt
[root@VM_133_231_centos test_tar]# tar -zcvf fout.tar.gz * --exclude=one_dir
three.txt
two_dir/
two_dir/two.txt
```

- tar, 解压指定目录
```sh

[root@VM_133_231_centos test_tar]# mkdir five_dir
[root@VM_133_231_centos test_tar]# tar -zxvf fout.tar.gz -C five_dir
three.txt
two_dir/
two_dir/two.txt
[root@VM_133_231_centos test_tar]# ll five_dir/
total 8
-rw-r--r-- 1 root root    6 Apr 14 08:47 three.txt
drwxr-xr-x 2 root root 4096 Apr 14 08:47 two_dir
```

- vim, 自动补齐功能

组合按钮|补齐的内容
-|-
[ctrl]+x --> [ctrl]+n |透过目前正在编辑的这个『 文件 的内容文字』作为关键词，予以补齐
[ctrl]+x --> [ctrl]+f |以当前目录内的『文件名』作为关键词，予以补齐
[ctrl]+x --> [ctrl]+o |以扩展名作为语法补充，以 vim 内建的关键词，予以补齐

- ${}, 替换的功能

变量设定方式|说明
-|-
${var/old_str/new_str}|若变量内容符合 old_str，则第一个旧字符串会被新字符串取代
${var//old_str/new_str}|若变量内容符合 old_str，则全部的旧字符串会被新字符串取代

```sh
[root@VM_133_231_centos ~]# echo ${test_str}    
abcabcabc
[root@VM_133_231_centos ~]# echo ${test_str/a/A}
Abcabcabc
[root@VM_133_231_centos ~]# echo ${test_str//a/A}
AbcAbcAbc

```

- alias, 设置别名
```sh
[root@VM_133_231_centos ~]# alias pg='ps -ef | grep'
[root@VM_133_231_centos ~]# alias
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias pg='ps -ef | grep'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
[root@VM_133_231_centos ~]# pg nginx
root      4922     1  0  2019 ?        01:20:59 node /usr/share/nginx/html/NodeJS-forum/bin/www
nginx    10308 30681  0  2019 ?        00:10:27 nginx: worker process
root     30573 30288  0 08:27 pts/0    00:00:00 grep --color=auto nginx
root     30681     1  0  2019 ?        00:00:00 nginx: master process /usr/sbin/nginx

```

- 将全部内容定向到指定文件，并且获取执行返回码

```sh
[root@VM_133_231_centos ~]# sh test.sh > output.txt 2>&1
[root@VM_133_231_centos ~]# echo $?
0
```

类型|代码|操作
-|-|-
标准输入 (stdin) |代码为 0 |使用 < 或 <<
标准输出 ( stdout)|代码为 1 |使用 > 或 >>
标准错误输出 ( stderr)|代码为 2 |使用 2> 或 2>>

- \|, 管道命令，传递标准输出 stdout, 也可追加错误输出
```sh
# stdout
[root@VM_133_231_centos ~]# ps -ef | grep nginx
root      1774  1719  0 07:09 pts/0    00:00:00 grep --color=auto nginx
nginx    10308 30681  0  2019 ?        00:10:31 nginx: worker process
root     30681     1  0  2019 ?        00:00:00 nginx: master process /usr/sbin/nginx

# stdout and stderr
[root@VM_133_231_centos ~]# cat not_exist_file 2>&1 | grep 'No such file or directory'
cat: not_exist_file: No such file or directory
[root@VM_133_231_centos ~]# cat not_exist_file 2>&1 | grep 'OK'
[root@VM_133_231_centos ~]# 

```


- && 和 \|\|

指令下达情况 |说明
-|-
cmd1 && cmd2|1. 若 cmd1 执行完毕且正确执行 ($?= 0)，则开始执行 cmd2 。<br> 2. 若 cmd1 执行完毕且为错误 ($?!=0)，则 cmd2 不执行。
cmd1 \|\| cmd2 | 1. 若 cmd1 执行完毕且正确执行 ($?= 0)，则 cmd2 不执行。<br> 2. 若 cmd1 执行完毕且为错误 ($?!=0)，则开始执行 cmd2 。



- cut, 字符串切割, -d 指定分隔符号, -f 指定列数
```sh
[root@VM_133_231_centos ~]# echo "a;b;c;d" | cut -d ";" -f 2,4
b;d

```

- sort, 指定列排序
```sh
[root@VM_133_231_centos ~]# cat test_file.txt 
1 jack
2 luse
3 mark
4 dog
5 cat
[root@VM_133_231_centos ~]# cat test_file.txt | sort -t " " -k 2
5 cat
4 dog
1 jack
2 luse
3 mark
```

- uniq, 去重并且计算出现次数
```sh
[root@VM_133_231_centos ~]# cat test_file.txt 
1 jack
2 luse
3 mark
4 dog
5 cat
6 mark
7 jack
8 mark
# 首先排序，然后截取字段，最后计算出现次数
[root@VM_133_231_centos ~]# cat test_file.txt | sort -t " " -k 2 | cut -d " " -f 2 | uniq -ic
      1 cat
      1 dog
      2 jack
      1 luse
      3 mark

```


- du, 查看当前目录文件大小，并且排序
```sh
[root@VM_133_231_centos local]# du -h --max-depth=1 | sort -h
4.0K	./bin
4.0K	./etc
4.0K	./games
4.0K	./include
4.0K	./lib
4.0K	./lib64
4.0K	./libexec
4.0K	./sbin
4.0K	./src
12K	    ./sa
92K	    ./share
14M	    ./hdf5
111M	./php56
191M	./gse
403M	./qcloud
734M	./python3
1.5G	.

```

- tee, 保存一份内容到文件，并且输出 stdout
```sh
# process.log 保存到文件
[root@VM_133_231_centos ~]# ps -ef | tee process.log | grep nginx 
root      4922     1  0  2019 ?        01:21:12 node /usr/share/nginx/html/NodeJS-forum/bin/www
root      8917  1719  0 07:54 pts/0    00:00:00 grep --color=auto nginx
nginx    10308 30681  0  2019 ?        00:10:31 nginx: worker process
root     30681     1  0  2019 ?        00:00:00 nginx: master process /usr/sbin/nginx

```

- split, 分割文件，按照大小或者行数
```sh
# 文件大小分割
[root@VM_133_231_centos ~]# split -b 2k process.log split_process_log_
[root@VM_133_231_centos ~]# ls -lh split_process_log*
-rw-r--r-- 1 root root 2.0K Apr 17 07:58 split_process_log_aa
-rw-r--r-- 1 root root 2.0K Apr 17 07:58 split_process_log_ab
-rw-r--r-- 1 root root 2.0K Apr 17 07:58 split_process_log_ac
-rw-r--r-- 1 root root 2.0K Apr 17 07:58 split_process_log_ad
-rw-r--r-- 1 root root  907 Apr 17 07:58 split_process_log_ae

# 行数分割
[root@VM_133_231_centos ~]# split -l 20 process.log split_process_line_
[root@VM_133_231_centos ~]# ls -l split_process_line*   
-rw-r--r-- 1 root root 1229 Apr 17 08:02 split_process_line_aa
-rw-r--r-- 1 root root 1245 Apr 17 08:02 split_process_line_ab
-rw-r--r-- 1 root root 1562 Apr 17 08:02 split_process_line_ac
-rw-r--r-- 1 root root 1546 Apr 17 08:02 split_process_line_ad
-rw-r--r-- 1 root root 1449 Apr 17 08:02 split_process_line_ae
-rw-r--r-- 1 root root 1387 Apr 17 08:02 split_process_line_af
-rw-r--r-- 1 root root  681 Apr 17 08:02 split_process_line_ag
[root@VM_133_231_centos ~]# cat split_process_line_aa | wc -l
20

```

- awk, 截取列值
```sh

[root@VM_133_231_centos ~]# ps -ef |grep php | head -n 10
root      1093     1  0  2019 ?        00:20:05 php-fpm: master process (/usr/local/php56/etc/php-fpm.conf)
apache    2317  1093  0  2019 ?        00:01:04 php-fpm: pool www
apache    2320  1093  0  2019 ?        00:01:05 php-fpm: pool www
apache    3181  8039  0 Apr10 ?        00:04:43 php-fpm: pool www
apache    3192  8039  0 Apr10 ?        00:04:41 php-fpm: pool www
apache    3193  8039  0 Apr10 ?        00:04:42 php-fpm: pool www
apache    3197  8039  0 Apr10 ?        00:04:42 php-fpm: pool www
apache    3198  8039  0 Apr10 ?        00:04:42 php-fpm: pool www
apache    3199  8039  0 Apr10 ?        00:04:42 php-fpm: pool www
apache    3200  8039  0 Apr10 ?        00:04:39 php-fpm: pool www
[root@VM_133_231_centos ~]# ps -ef |grep php | head -n 10 | awk '{print $2}'
1093
2317
2320
3181
3192
3193
3197
3198
3199
3200

```

- xargs, 返回参数形式
```sh
[root@VM_133_231_centos ~]# ps -ef |grep php | head -n 10 | awk '{print $2}' | xargs 
1093 2317 2320 3181 3192 3193 3197 3198 3199 3200

```

- 杀死指定名称的所有进程
```sh
[root@VM_133_231_centos ~]# ps -ef | grep php | grep -v grep | awk '{print $2}' | xargs kill -9
```

- 修改时区
```sh

[root@VM_133_231_centos ~]# timedatectl 
      Local time: Sun 2020-04-19 08:15:51 CST
  Universal time: Sun 2020-04-19 00:15:51 UTC
        RTC time: Sun 2020-04-19 00:15:44
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
[root@VM_133_231_centos ~]# timedatectl list-timezones | grep -i new_york  
America/New_York
[root@VM_133_231_centos ~]# timedatectl set-timezone America/New_York
[root@VM_133_231_centos ~]# timedatectl 
      Local time: Sat 2020-04-18 20:17:26 EDT
  Universal time: Sun 2020-04-19 00:17:26 UTC
        RTC time: Sun 2020-04-19 00:17:19
       Time zone: America/New_York (EDT, -0400)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: yes
 Last DST change: DST began at
                  Sun 2020-03-08 01:59:59 EST
                  Sun 2020-03-08 03:00:00 EDT
 Next DST change: DST ends (the clock jumps one hour backwards) at
                  Sun 2020-11-01 01:59:59 EDT
                  Sun 2020-11-01 01:00:00 EST
```