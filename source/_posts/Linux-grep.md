---
title: Linux grep
date: 2020-04-18 07:18:45
tags: Linux
---

> grep 

<!-- more -->

- 根据关键字查找
```sh
[root@VM_0_10_centos ~]# grep "a" test.txt 
1. apple
2. football
3. Jack
6. cat
```

- 利用正则查找范围内的字符串
```sh
[root@VM_0_10_centos ~]# grep "[1-3]" test.txt 
1. apple
2. football
3. Jack

[root@VM_0_10_centos ~]# grep "a[a-m]" test.txt 
2. football
3. Jack

```


- 正则去掉指定开头的字符串
```sh
[root@VM_0_10_centos ~]# grep "[^b]a" test.txt 
1. apple
3. Jack
6. cat
```

- 正则查找指定开头的字符串
```sh
[root@VM_0_10_centos ~]# grep "^3" test.txt 
3. Jack
```
- 正则查找指定范围开头的字符串
```sh
[root@VM_0_10_centos ~]# grep "^[3-5]" test.txt 
3. Jack
4. Mike
5. dog
```

- 正则指定结尾的字符串
```sh
[root@VM_0_10_centos ~]# grep "e$" test.txt 
1. apple
4. Mike
[root@VM_0_10_centos ~]# grep "[a-m]$" test.txt 
1. apple
2. football
3. Jack
4. Mike
5. dog

```

- 找出空行
```sh
[root@VM_0_10_centos ~]# cat test.txt 

1. apple

2. football
3. Jack

4. Mike
5. dog
6. cat
[root@VM_0_10_centos ~]# grep "^$" test.txt 



[root@VM_0_10_centos ~]# grep "^$" test.txt  |wc -l
3

```
