---
title: JS 驼峰和下横线转换
date: 2019-10-26 12:17:14
tags: JavaScript
---

> 利用正则批量下横线和大写字符，然后进行转换

<!-- more -->


```js
//下横线转驼峰
function underline2Hump(s) {
    // 连续下横线的情况也要兼容
    var r = s.replace(/_+(\w)/g, function(all, letter) {
        return letter.toUpperCase()
    });
    // 第一个字符是下横线，会导致首字母是大写，这里改成小写
    r=r.slice(0,1).toLowerCase()+r.slice(1);
    return r;
}

//驼峰转下横线：
function hump2Underline(s) {
    // 连续大写字母的情况也兼容
    var r=s.replace(/([A-Z])/g, '_$1').toLowerCase();
    // 首字母大写，会导致第一个字符也是下横线，需要去掉
    if(r.slice(0,1) === '_'){
        r=r.slice(1);
    }
    return r;
}

console.log(underline2Hump('_one__two___three'));
console.log(hump2Underline('OneTwoTHRee'));

```

### [源代码](/example/js/hump-and-underline.html)