---
title: 二分法查找
date: 2019-10-22 21:18:00
tags: algorithm
---

> 二分法查找，在有序数组中查找特定元素的搜索算法，时间复杂度是 O(logn)

<!-- more -->

### 一，二分法查找原理：
##### 1.二分法查找，在有序数组中查找特定元素的搜索算法，时间复杂度是 O(logn)；
##### 2.获取数组中间位置的数值进行对比，判断搜索元素在小区间还是大区间，如果在大区间就继续分割大区间的数组，一直找到元素或者不能再分割为止；小区间也是一样的分割查找方法

### 二，JS 实现二分法查找：

```js
function binarySearch(arr,num){
    var find_index = 0;
    var find_arr = arr.slice();
    var arr_len = find_arr.length;
    var mid_index = 0;

    // 必须在有序数组的区间内，否则报错
    if(num < arr[0]){
        return -1;
    }
    if(num > arr[arr_len-1]){
        return -1;
    }

    // 一直分割查找到数组到单个元素为止
    while( arr_len > 1){
        mid_index = Math.floor(arr_len/2);
        // 在较大的区间
        // 等于也划分到大区间，所以有序数组出现两个相同的数值时，找到的是最后一个
        if(num>=find_arr[mid_index]){
            // 下标累加
            find_index = find_index + mid_index;
            // 截取较大区间数组
            find_arr = find_arr.slice(mid_index);
        }else{
            // 截取较小区间数组
            find_arr = find_arr.slice(0,mid_index);
        }
        arr_len = find_arr.length;
    }

    // 找到了
    if(find_arr[0]==num){
        // 是否存在两个相同的数值，获取第一个
        if( find_index > 0 && arr[find_index-1]==num){
            return find_index-1;
        }
        return find_index;
    // 没找到
    }else{
        return -1;
    }
}

```


### [源代码](/example/js/binary_search.html)


### 三，二分法查找关键点

1. 计算 mid 时需要防止溢出，代码中 left + (right - left) / 2 和 (left + right) / 2 的结果相同，但是前者有效防止了 left 和 right 太大直接相加导致溢出。

2. 当 right - left + 1 等于偶数时，比如 0 1 2 3 4 5 ，计算 mid 可能会取 2(左中位)，或者取 3(右中位)，如果取左中位，那么 mid = 0 + (5 - 0)/2 = 2，即 mid = left + (right - left )/2 ；如果取右中位，那么 mid = 0 + (5 - 0 + 1)/2 = 3，即 mid = left + (right - left + 1)/2
