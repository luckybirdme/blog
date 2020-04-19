---
title: Linux shell
date: 2020-04-18 11:33:10
tags: Linux
---

> shell

<!-- more -->


- test 用来判断真假，也可以用中括号[], 括号两边必须有空格
```sh

[root@VM_0_10_centos ~]# test -z ""; echo $?
0
[root@VM_0_10_centos ~]# test -z "a"; echo $?
1
[root@VM_0_10_centos ~]# [ -z "" ]; echo $?
0
[root@VM_0_10_centos ~]# [ -z "a" ]; echo $?
1
[root@VM_0_10_centos ~]# cat test.sh 
p='a'
if [ -z $p ];then
	echo 'z'
else
	echo '!z'
fi
if test -z $p;then
        echo 'z'
else
        echo '!z'
fi
[root@VM_0_10_centos ~]# sh test.sh 
!z
!z
```

- 文件判断

命名|含义
-|-
-e |该『档名』是否存在？(常用) 
-f |该『档名』是否存在且为文件(file)？(常用) 
-d |该『文件名』是否存在且为目录(directory)？(常用)
-r |侦测该档名是否存在且具有『可读』的权限？
-w |侦测该档名是否存在且具有『可写』的权限？
-x |侦测该档名是否存在且具有『可执行』的权限？

- 数值判断

命名|含义
-|-
-eq |两数值相等 (equal)
-ne |两数值不等 (not equal)
-gt |n1 大于 n2 (greater than)
-lt |n1 小于 n2 (less than)
-ge |n1 大于等于 n2 (greater than or equal)
-le |n1 小于等于 n2 (less than or equal)


- 变量判断

命名|含义
-|-
-z string |若 string 为空字符串，则为 true
-n string | 若 string 为空字符串，则为 false。注： -n 亦可省略
str1 == str2 |判定 str1 是否等于 str2 ，若相等，则回传 true
str1 != str2 |判定 str1 是否不等于 str2 ，若相等，则回传 false


- 变量判断

命名|含义
-|-
\-a|(and)两状况同时成立！例如  -r file -a -x file，则 file 同时具有 r 与 x 权限时，才回传 true。 
\-o|(or)两状况任何一个成立！例如  -r file -o -x file，则 file 具有 r 或 x 权限时，就可回传 true。 
\!| 反相状态，如  ! -x file ，当 file 不具有 x 时，回传 true


- 接收参数

参数处理|	说明
-|-
\$#|	传递到脚本的参数个数
\$*|	以一个单字符串显示所有向脚本传递的参数。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
\$\$|	脚本运行的当前进程ID号
\$!|	后台运行的最后一个进程的ID号
\$@|	与$*相同，但是使用时加引号，并在引号中返回每个参数。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。
\$-	|显示Shell使用的当前选项，与set命令功能相同。
\$?|	显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。

- case 使用
```sh
[root@VM_133_231_centos ~]# cat test.sh      
#!/bin/bash
input=$1
case $input in 
    "start") 
        echo "${input} process" 
        ;;
    "") 
        echo "You MUST input : start|stop|reload" 
        ;;
    # 通配符
    *)
        echo "Usage ${0} : sh ${0} start " 
        ;;
esac

[root@VM_133_231_centos ~]# sh test.sh start
start process
[root@VM_133_231_centos ~]# sh test.sh      
You MUST input : start|stop|reload
[root@VM_133_231_centos ~]# sh test.sh dfdsafd
Usage test.sh : sh test.sh start 

```


- 函数使用
```sh
[root@VM_133_231_centos ~]# cat test.sh 
#!/bin/bash
function log(){
    date_str=`date +"%Y-%m-%d %H:%M:%S"`
    log_content="${date_str} $*"
    echo ${log_content} >> log_file.txt
}

log "1"
log "2"
log "3"
[root@VM_133_231_centos ~]# sh test.sh 
[root@VM_133_231_centos ~]# cat log_file.txt 

2020-04-18 11:25:23 1
2020-04-18 11:25:23 2
2020-04-18 11:25:23 3

```


- while 循环
```sh
[root@VM_133_231_centos ~]# cat test.sh 
#!/bin/bash
i=0
while [ $i -lt 3 ]
do
	let i++  
	echo $i
	sleep 1
done
[root@VM_133_231_centos ~]# sh test.sh 
1
2
3
```

- for 循环
```sh

[root@VM_133_231_centos ~]# cat test.sh 
#!/bin/bash
str='one two three'
for i in $str
do
	echo $i
done
[root@VM_133_231_centos ~]# sh test.sh 
one
two
three
```

