---
title: php 排序算法
date: 2020-04-29 07:21:17
tags: algorithm
---

> 冒泡，选择，插入，希尔，归并，快速

<!-- more -->

### 冒泡排序
1. 从第一个元素开始，与右边第二的元素比较，如果比右边元素大(小)，则互换位置；然后第二元素继续与右边第三个元素比较，如果比右边元素小(大)，则互换位置。
2. 以此类推，将较大(小)元素一直往右边移动，类似气泡一样往上冒泡，第一次将最大(小)值移到倒数第一个位置；第二次将次大(小)移到倒数第二位置，那么冒泡 n - 1 次后，就有序了。

```php
$arr = array(8,3,5,2,4,1,6,7,9);
function bubbleSort($arr){
    // 数组长度，遍历的停止条件
    $len = count($arr);
    // 临时值，用于交换元素
    $tmp = '';
    // 需要冒泡 n - 1 次
    for ($i=0; $i < $len - 1 ; $i++) {
        // 从第一个元素开始，与右边第二元素比较，判断是否需要互换位置
        for ($j=0; $j < $len - 1 - $i; $j++) { 
            // 如果当前元素比右边的元素大，则往右边移动
            if($arr[$j]>$arr[$j+1]){
                // 将当前元素和右边元素的位置互换
                $tmp = $arr[$j+1];
                $arr[$j+1]=$arr[$j];
                $arr[$j]=$tmp;
            }
        }
    }
    return $arr;
}
print_r(bubbleSort($arr));
```

### 选择排序
1. 从左边第一个元素开始，与右边剩下的所有元素对比，得出最小(大)的元素，将第一个元素与最小(大)元素的位置互换
2. 接着从左边第二个元素开始，与右边元素对比得出最小(大)元素，然后第二个元素与最小(大)元素位置互换，以此类推，直到最后一个元素
3. 由于位置互换，相等元素的顺序可能会打乱，所以选择排序是不稳定的，比如 5 3 5 2 9，第一个 5 将会与 2 互换位置，那这两个 5 的原本顺序就打乱了。
4. 如果将最小值换到前面，那就是顺序，如果最大值换到前面，那就是倒序。

```php
$arr = array(8,3,5,2,4,1,6,7,9);

function selectSort($arr){
    // 数组长度，遍历的停止条件
    $len = count($arr);
    // 最小元素的下标
    $min = '';
    // 交换元素时的临时值
    $tmp = '';
    // 从第一个元素开始，与右边元素比较
    for ($i=0; $i < $len - 1; $i++) {
        // 默认当前元素是最小的，并且记录下标
        $min = $i;
        // 与右边剩下元素比较，找出最小值，并且记录下标
        for ($j=$i+1; $j < $len - 1; $j++) { 
            // 如果有更小值，则把下标记录下来
            if($arr[$j]<$arr[$min]){
                // 保存最小元素的下标
                $min = $j;
            }
        }
        // 将当前元素和最小元素的位置互换
        // 每次都将最小元素换到前面，就能达到排序的效果
        $tmp = $arr[$i];
        $arr[$i] = $arr[$min];
        $arr[$min] = $tmp;
    }
    return $arr;
}

print_r(selectSort($arr));

```


### 插入排序
1. 类似打扑克，将牌插入到合适位置，比前一个元素大， 比后一个元素小，当所有牌都按照以上情况插入后，就有顺序了。
2. 从第二元素开始，跟左边的元素对比，如果比当前元素大，则将左边元素往右边移动一个位置，以此类推，移动左边剩下的元素。
3. 直到左边的元素比当前元素小，右边的元素比当前元素大，这就是当前元素插入的位置。

```php

$arr = array(8,3,5,2,4,1,6,7,9);
function insertSort($arr){
    // 数组长度，遍历的停止条件
    $len = count($arr);
    // 插入位置的下标
    $int = '';
    // 需要插入的元素值
    $tmp = '';
    // 从第二元素开始，跟左边的元素对比，找到比前一个元素大，比后一个元素小的位置
    for ($i=1; $i < $len - 1; $i++) { 
        // 当前元素值，即将要插入的元素值，避免被覆盖
        $tmp = $arr[$i];
        // 设置默认插入的位置，就是本身，即不需要变动
        $int = $i;
        // 往左边的元素进行对比，直到最左边的元素，即下标为 0 的元素
        for ($j=$i-1; $j >= 0; $j--) { 
            // 如果左边的元素比当前元素大，则将左边的元素往右移一个位置
            if($arr[$j]>$tmp){
                $arr[$j+1] = $arr[$j];
                $int = $j;
            }else{
                // 如果没有比当前元素更大的，说明当前元素应该插入到这个位置
                break;
            }
        }
        // 将当前元素插入到上面找到的位置
        $arr[$int] = $tmp;

    }
    return $arr;
    
}

print_r(insertSort($arr));

```


### 希尔排序
1. 希尔排序是插入排序的升级版。
2. 原理请参考[这里](http://blog.luckybird.me/2019/10/19/my-algorism-func/)


```php

$arr = array(8,3,5,2,4,1,6,7,9);
function shellSort($arr){
    // 数组长度，遍历的停止条件
    $len = count($arr);
    // 插入位置的下标
    $int = '';
    // 需要插入的元素值
    $tmp = '';

    // 分组数
    $gap = $len;
    // 分组数为 1 时，相当于进行一次全员插入排序
    while($gap>0){
        // 右移一步，相当于除以 2，所以每次分组数减半
        $gap=$gap>>1;
        // 从第二元素开始，跟左边的元素对比，找到比前一个元素大，比后一个元素小的位置
        for($i=$gap; $i<$len-1; $i++) {
            // 当前元素值，即将要插入的元素值，避免被覆盖
            $tmp = $arr[$i];
            // 设置默认插入的位置，就是本身，即不需要变动
            $int = $i;
            // 往左边的元素进行对比，直到最左边的元素，即下标为 0 的元素
            for($j=$i-$gap;$j>=0;$j=$j-$gap){
                // 如果左边的元素比当前元素大，则将左边的元素往右移一个位置
                if($arr[$j]>$tmp){
                    $arr[$j+$gap] = $arr[$j];
                    $int = $j;
                }else{
                    // 如果没有比当前元素更大的，说明当前元素应该插入到这个位置
                    break;
                }
                    
            }
            // 将当前元素插入到上面找到的位置
            $arr[$int] = $tmp;

        }

    }
    return $arr;
    
}

print_r(shellSort($arr));

```


### 归并排序


```php

// 合并两个数组
function mergeArr($left_arr,$right_arr){
    // 合并后的新数组
    $result = array();

    // 遍历两个数组的同位元素，比较大小，按照顺序合并
    while (count($left_arr)>0 && count($right_arr) >0) {
        if($left_arr[0]<$right_arr[0]){
            $result[]=array_shift($left_arr);
        }else{
            $result[]=array_shift($right_arr);
        }
    }

    // 如果其中一个数组还有元素，那肯定比较大，所以直接追加到新数组后面
    if(count($left_arr)>0){
        $result=array_merge($result,$left_arr);
    }
    if(count($right_arr)>0){
        $result=array_merge($result,$right_arr);
    }

    return $result;
}

// 递归调用
function mergeSort($arr){
    // 二分法拆分数组，直到数组为单个元素
    $len = count($arr);
    if($len<2){
        return $arr;
    }
    
    // 两个拆分出来的临时数组
    $left_arr = array();
    $right_arr = array();

    // 右移，相当于除以 2 ，二分法
    $middle = $len>>1;
    $left_arr = array_slice($arr, 0, $middle);
    $right_arr = array_slice($arr, $middle);

    // 递归拆分数组，然后合并数组
    return mergeArr(mergeSort($left_arr),mergeSort($right_arr));
    
}

$arr = array(8,3,5,2,4,1,6,7,9);

print_r(mergeSort($arr));

```


### 快速排序

1. 从数列中挑出一个元素，称为 “基准”（pivot）;
2. 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面，相同的数可以到任一边，所以快速排序是不稳定的。
3. 递归地把小于基准值元素的子数列和大于基准值元素的子数列进行上述一样的排序操作，直到子数列元素为一个为止。
4. 递归的最底部情形，数列元素已经被排序好了，然后将所有子数列合并，那么数列元素就有序了。

```php

function quicktSort($arr){
    // 数组长度，遍历的停止条件
    $len = count($arr);
    if($len<2){
        return $arr;
    }
    // 分割基准，选择第一个元素
    $pivot = $arr[0];

    // 小于基准的数组
    $left_arr = array();
    // 大于基准的数组
    $right_arr = array();

    // 从第二元素开始，跟基准对比
    for ($i=1; $i < $len; $i++) {
        // 大于基准的，放在右边数组，否则放在左边 
        if($arr[$i]>$pivot){
            $right_arr[]=$arr[$i];
        }else{
            $left_arr[]=$arr[$i];
        }
    }
    // 继续分割数组，直到只剩下一个元素
    // 最后将数组合并起来，就是有序的数组了
    return array_merge(quicktSort($left_arr),array($pivot),quicktSort($right_arr));
    
}

$arr = array(1,3,5,2,4,8,6,7,9);

print_r(quicktSort($arr));
```



### 堆排序

```php

function swap(&$x,&$y){
    $t=$x;
    $x=$y;
    $y=$t;
}

// 调整父节点，建立大顶堆
function heapify(&$arr, $root,$len){
    $left=$root*2+1;
    $right=$root*2+2;
    
    // 左子节点必须存在
    if($left>=$len){
        return false;
    }
    
    // 默认左子节点最大
    $son = $left;
    // 如果存在右节点，而且比较大
    if($right<$len && $arr[$right]>$arr[$left]){
        $son=$right;
    }

    if($arr[$son]>$arr[$root]){
        swap($arr[$root],$arr[$son]);
        // 遍历子节点下的子子节点
        heapify($arr,$son,$len);
    }
}

function heapSort($arr){
    $len = count($arr);
    // 遍历每个父节点，构建大顶堆
    for($i=floor(($len-1)/2);$i>=0;$i--){
        heapify($arr,$i,$len);
    }

    // 将大顶堆的值和数组末尾值交互，达到排序的效果
    for($i=$len-1;$i>0;$i--){
        //print_r($arr);
        swap($arr[0],$arr[$i]);
        // 再次从顶部开始，构建大顶堆
        heapify($arr,0,$i);
    }

    return $arr;
}

$arr = array(8,3,5,2,4,1,6,7,9);
print_r(heapSort($arr));

```