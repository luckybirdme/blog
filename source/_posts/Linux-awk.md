---
title: Linux awk
date: 2020-04-18 07:21:05
tags: Linux
---

> awk

<!-- more -->


- 特殊变量

变量名称|代表意义
-|-
NF |每一行 ($0) 拥有的字段总数
NR |目前 awk 所处理的是『第几行』数据
FS |目前的分隔字符，默认是空格键

```sh
[root@VM_0_10_centos ~]# cat test.txt 
1. apple banana
2. football
3. Jack lose
4. Mike
5. dog bird
6. cat mose fly
[root@VM_0_10_centos ~]# cat test.txt |  awk '{print $1 "\t lines: " NR "\t columns: " NF}'
1.	 lines: 1	 columns: 3
2.	 lines: 2	 columns: 2
3.	 lines: 3	 columns: 3
4.	 lines: 4	 columns: 2
5.	 lines: 5	 columns: 3
6.	 lines: 6	 columns: 4

```

- 指定分割符号
```sh
[root@VM_0_10_centos ~]# cat test.txt |  awk 'BEGIN {FS="."}  {print $1 "\t lines: " $2}'
1	 lines:  apple banana
2	 lines:  football
3	 lines:  Jack lose
4	 lines:  Mike
5	 lines:  dog bird
6	 lines:  cat mose fly

```

- 使用条件帅选
```sh
[root@VM_0_10_centos ~]# cat test.txt |  awk 'BEGIN {FS="."} $1 > 4 {print $1 "\t lines: " $2}'
5	 lines:  dog bird
6	 lines:  cat mose fly
# NR 是当前行数
[root@VM_0_10_centos ~]# cat test.txt |  awk 'BEGIN {FS="."} NR == 4 {print $1 "\t lines: " $2}'
4	 lines:  Mike

```
