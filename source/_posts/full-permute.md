---
title: JS 输出全排列
date: 2019-10-24 06:53:52
tags: JavaScript
---

> 从n个不同元素中任取m（m≤n）个元素，按照一定的顺序排列起来，叫做从n个不同元素中取出m个元素的一个排列。当m=n时所有的排列情况叫全排列

<!-- more -->

### 一，全排列定义：
##### 从n个不同元素中任取m（m≤n）个元素，按照一定的顺序排列起来，叫做从n个不同元素中取出m个元素的一个排列。当m=n时所有的排列情况叫全排列。

### 二，输出全排列的方法：

##### 1. DFS + 链接法
- DFS，即深度遍历优先(Depth-First Search)，先从第一个元素开始，一直遍历到最末尾的元素为止，之后，再遍历兄弟节点的元素，如下图所示

![](/img/2019/full-permute-0.jpg)

- 链接法是指将一个个元素，链接到末尾，以达到排列的效果，对于不重复的数组的全排列效果图如下

![](/img/2019/full-permute-1.jpg)

##### 2. 代码实现

```js
function full_permute(arr){
    // 存放全排列的数组
    var result = [];
    var main = function(input){
        // 如果数组长度等于原始长度，说明排列好了一个啦
        if(input.length==arr.length){
            result.push(input);
            return;
        }
        // 从第一个数开始排列
        for(var i in arr){
            // 如果元素不在数组里，则把元素链接到数组末尾，达到排列的效果
            if(input.indexOf(arr[i]) === -1){
                // 通过 concat 复制和添加元素，以便深克隆数组
                // 避免只是传递数组引用，影响原有数组
                // 递归
                main(input.concat(arr[i]));
            }
        }
    }
    // 开始排列
    main([]);
    return result;
}

```
### [源代码](/example/js/full-permute.html)


