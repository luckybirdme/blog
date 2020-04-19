---
title: Linux 用户管理
date: 2020-04-18 11:37:20
tags: Linux
---

> user

<!-- more -->


- 用户信息查询

```sh
[root@VM_133_231_centos ~]# cat /etc/passwd | head -n 3
# 格式意义
# 用户名:用户密码:用户ID:用户组ID:用户说明:用户家目录:用户登录控制
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
[root@VM_133_231_centos ~]# cat /etc/group | head -n 3
# 用户组名称:用户密码:用户组ID
root:x:0:
bin:x:1:
daemon:x:2:
```

- 创建用户组
```sh
[root@VM_133_231_centos ~]# groupadd test_group -g 10000
[root@VM_133_231_centos ~]# cat /etc/group | grep test_group
test_group:x:10000:
```

- 创建用户
```sh
[root@VM_133_231_centos ~]# useradd test_user_four -u 10004 -g 10000 -d /home/test_user_four -c test_add_user_four
[root@VM_133_231_centos ~]# cat /etc/passwd |grep test_user_four
test_user_four:x:10004:10000:test_add_user_four:/home/test_user_four:/bin/bash
```