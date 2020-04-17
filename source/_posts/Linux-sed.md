---
title: Linux sed
date: 2020-04-18 07:18:52
tags: Linux
---

> sed

<!-- more -->


- 删除指定行的内容
```sh
[root@VM_0_10_centos ~]# cat test.txt 
1. apple
2. football
3. Jack
4. Mike
5. dog
6. cat
[root@VM_0_10_centos ~]# cat test.txt | sed '2,5d'
1. apple
6. cat

```

- 在指定行后面追加内容
```sh
[root@VM_0_10_centos ~]# cat test.txt | sed '2a add in two line'
1. apple
2. football
add in two line
3. Jack
4. Mike
5. dog
6. cat

```

- 在指定行前面插入内容
```sh
[root@VM_0_10_centos ~]# cat test.txt | sed '2i inset in two line'
1. apple
inset in two line
2. football
3. Jack
4. Mike
5. dog
6. cat
```

- 替换指定行内容
```sh
[root@VM_0_10_centos ~]# cat test.txt | sed '2c replace in two line'
1. apple
replace in two line
3. Jack
4. Mike
5. dog
6. cat
```

- 帅选指定行, 参数 n 是采用静默模式，避免 stdout 影响
```sh
[root@VM_0_10_centos ~]# cat test.txt | sed -n '2,5p'
2. football
3. Jack
4. Mike
5. dog

```

- 替换指定字符串内容
```sh
[root@VM_0_10_centos ~]# cat test.txt | sed  's/dog/DOG/g'
1. apple
2. football
3. Jack
4. Mike
5. DOG
6. cat
```

- 直接文件内容，这个需要慎重
```sh
[root@VM_0_10_centos ~]# sed -i 's/dog/DOG/g' test.txt 
[root@VM_0_10_centos ~]# cat test.txt 
1. apple
2. football
3. Jack
4. Mike
5. DOG
6. cat
```

