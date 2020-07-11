---
title: JS 实现阶乘函数
date: 2019-10-23 08:35:30
tags: algorithm
---

> 阶乘原理

<!-- more -->

### 一，阶乘原理
|n取值|运算规则|
|-|-|
|<0|-1|
|0|1|
|>=1| n * (n -1) * (n - 2) ... 3 * 2 * 1|

### 二，JS 实现

```js
function factorial(num){
    if(num<0){
        return -1;
    }
    if(num==0){
        return 1;
    };

    var rtn=num;
    while(num>1){
        num--;
        rtn=rtn*num;
    };
    return rtn;
}

```
