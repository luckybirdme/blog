---
title: Linux crontab
date: 2020-04-18 12:10:44
tags: Linux
---

> crontab

<!-- more -->


- 格式

```sh
5 1 * * 1 sh test.sh

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

```


- 特殊字符串意义

特殊字符|代表意义
-|-
星号(*)|代表任何时刻都接受的意思！举例来说，范例一内那个日、月、周都是 * 就代表着『不论何月、何日的礼拜几的 12:00 都执行后续指令』的意思！
逗号(,) | 代表分隔时段的意思。举例来说，如果要下达的工作是 3:00 与 6:00 时，就会是：0 3,6 * * * command 时间参数还是有五栏，不过第二栏是 3,6 ，代表 3 与 6 都适用！
减号(-)|代表一段时间范围内，举例来说， 8 点到 12 点之间的每小时的 20 分都进行一项工作：20 8 12 * * * command 仔细看到第二栏变成 8 12 喔！代表 8,9,10,11,12 都适用的意思！
斜线(/)|那个 n 代表数字，亦即是『每隔 n 单位间隔』的意思，例如每五分钟进行一次，则：\*/5 * * * * command 很简单吧！用 * 与 /5 来搭配，也可以写成 0 59/5 ，相同意思
