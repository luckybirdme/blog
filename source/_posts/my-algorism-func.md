---
title: JS 实现基本排序算法
date: 2019-10-19 15:04:23
tags: JavaScript
---

> 常用的基本排序算法包括冒泡排序，选择排序，插入排序，归并排序，快速排序。
> 算法底层原理一致，不同语言实现方式不一样，下来用 JavaScript 实现以上排序算分。

<!-- more -->
## 排序算法的度量
### 一，时间复杂度
- 算法时间复杂度用 O(x) 表示，x 是数组长度 n 的度量函数，每个算法不一样。
- 对于长度为 n 的数组，排序只用一层 for 循环，那么 x=n，即这个排序算法的时间复杂度是 O(n)。 
- 那么两层 for 循环就等于 n 乘以 n，即 x=n^2；利用二分法的 for 循环，那么 x=nlogn。

### 二，空间复杂度
- 算法空间复杂度是指使用内存的情况，函数表示 O(x)，跟时间复杂度一样，x 是数组长度 n 的度量函数。
- 如果使用的内存数为常量，那么 x=1；如果会复制整个数组，那么 x=n；二分法循环复制，那么 x=logn

### 三，稳定性
- 原始数组的两个元素值相等，在排序后，两个元素相对的前后顺序是否会变化。
- 比如数组 [3,2(第一个),4,2(第二个),1]，第一个 2 和 第二 2 的相对位置，在排序没有变化，即 [1,2(第一个),2(第二个),3,4]，那么这个算法是稳定。否则就是不稳定。

### 四，常用排序算法的情况
|排序算法|平均时间复杂度|空间复杂度|稳定性|
|-|-|-|-|
|冒泡排序|O(n^2)|O(1)|稳定|
|选择排序|O(n^2)|O(1)|不稳定|
|插入排序|O(n^2)|O(1)|稳定|
|归并排序|O(nlogn)|O(n)|稳定|
|快速排序|O(nlogn)|O(logn)|不稳定|



## 基本原理和实现
### 一，冒泡排序
##### 原理：
1. 冒泡，顾名思义，将较大的数值一直往后冒泡，达到排序的效果。
2. 从第一个数开始，比较相邻（即第二个数）两个数值大小，较大值往右边放，较小值往左边放。
3. 以此类推，直到倒数第二个数和倒数第一个数比较为止，经过第一轮的遍历比较，最大值就会出现在最右边了。
4. 再次重复步骤 1，直到倒数第三个数和倒数第二个数比较为止，经过第二轮的遍历比较，次大值就会出现倒数第二个位置。
5. 按照以上步骤，对于长度为 n 的数组，经过 n-1 轮遍历比较，就可以完成排序

##### 动图：
![](/example/img/bubbleSort.gif)


##### 实现：
```js
// 冒泡
function bubbleSort(arr){
	var len = arr.length;
	var tmp = null;
	for(var i=0;i<len-1;i++){
		for(var j=0;j<len-1-i;j++){
			// 两个相邻的数组值进行比较
			// 把较大的数值一直往后冒泡
			if(arr[j]>arr[j+1]){
				tmp=arr[j+1];
				arr[j+1]=arr[j];
				arr[j]=tmp;
			}
		}
	}
}
```


### 二，选择排序
##### 原理：
1. 从左边数值开始往右比较，寻找最小值，跟左边对应数值位置对调。
2. 从第一个数值开始，跟第二，三 ··· 到最后一个数值比较，找到最小值的位置，然后跟第一个数值对调，这时候最小值就出现在最左边了。
3. 以此类推，从第二个数值开始，逐一往后比较，找到最小值跟第二个数值对调，次小值就会出现在第二个位置。
4. 按照以上步骤，对于长度为 n 的数组，经过 n-1 轮遍历比较，就可以完成排序

##### 动图：
![](/example/img/selectSort.gif)


##### 实现：
```js
// 选择
function selectSort(arr){
	var len = arr.length;
	var tmp = null;
	var min = 0;
	for(var i=0;i<len-1;i++){
		min=i;
		// 寻找最小的数值的小标
		for(var j=i+1;j<len;j++){
			if(arr[min]>arr[j]){
				min=j;
			}
		}
		// 将最小数对调到前面
		// 保存最小的数值
		tmp = arr[min];
		// 将最小数值的位置替换成前面的数
		arr[min] = arr[i];
		// 将最小的数值赋给前面的位置。
		arr[i] = tmp;
	}
}
```


### 三，插入排序
##### 原理：
1. 类似玩扑克牌，把数字牌按照大小插入到对应位置。
2. 从第二个数值开始，跟左边所有数值逐一比较，如果左边的数值比第二个数值大，则左边的数值往右边移一个位置；如果小于第二个数值，则把第二个数值插到这个数值后面。
3. 以此类推，从第三个数值开始，跟左边所有数值逐一比较，在小于第三个数值的位置后面插入第三个数值。
4. 按照以上步骤，对于长度为 n 的数组，经过 n-1 轮遍历比较，就可以完成排序。


##### 动图：
![](/example/img/insertSort.gif)


##### 实现：
```javascript
// 插入
function insertSort(arr){
	var len = arr.length;
	var tmp = null;
	var insert = 0;
	for(var i=1;i<len;i++){
		tmp=arr[i];
		insert=i-1;
		// 寻找合适的插入位置
		// 较大的数值依次完后移动
		while(insert>=0 && arr[insert]>tmp){
			arr[insert+1]=arr[insert];
			insert--;
		}
		// 插入到具体位置
		arr[insert+1]=tmp;
	}
}
```


### 四，归并排序
##### 原理：
1. 归并排序首先是按照二分法递归分割数组，直到数组变成单个元素，两个单元素的数组比较排序，之后再合并。
2. 每次合并后的数组元素都已排好顺序，递归以上操作，最后合并成一个排好顺序的数组。


##### 动图：
![](/example/img/mergeSort.gif)



##### 实现：
```javascript

// 归并
function mergeSort(arr){
	var merge=function(left,right){
		var result = [];

		// 对比两个数组相同下标的数组大小
		// 按照大小排序，生成新的数组
		while(left.length > 0 && right.length > 0){
			if(left[0]<=right[0]){
				result.push(left.shift());
			}else{
				result.push(right.shift());
			}
		}

		// 如果
		while(left.length > 0){
			result.push(left.shift());
		}

		while(right.length > 0){
			result.push(right.shift());
		};
		return result;
	}

	var slice=function(arr){
		var len = arr.length;
		if(len<2){
			return arr;
		}

		// 分割数组
		var middle = Math.floor(len/2);
		var left = arr.slice(0,middle);
		var right = arr.slice(middle);
		// 递归分割
		// 递归合并
		return merge(slice(left),slice(right));
	};

	return slice(arr);
}

```


### 五，快速排序
##### 原理：
1. 快速排序是冒泡排序和归并排序的升级版。
2. 冒泡排序比较的是相邻的数值大小，所以不同数值之间都有可能比较多次。
3. 快速排序首先选定一个基准参数来分割数组，把小于基准参数的元素放在左边数组，大于基准参数的元素放在右边数组。
4. 以此类推，左边数组再次选定一个基准参数来分割数组，直到数组分割成单元素，此时返回就是排好的顺序数组。
5. 利用递归分割数组，再把左右两个数组和基准参数合并成一个数组，就是一个排好顺序的数组。
6. 因为按照基准参数分割了数组，数组再次分割比较的时候，更小的数值只会跟更小的比较，不需要跟更大数值比较，所以比较次数少于冒泡排序，这也是快速排序优势。

##### 动图：

![](/example/img/quickSort.gif)

##### 实现：
```javascript
// 快速
function quickSort(arr){
	var quick = function(arr){
		var len = arr.length;
		if(len<2){
			return arr;
		};

		// 以第一个数值为基准值
		var base = arr.shift();
		// 减少长度 1，避免溢出
		len = len - 1;
		var left=[];
		var right=[];
		
		// 将数组分割
		// 大于基准值的放在右边数组
		// 小于基准值的放在左边数组
		for(var i=0;i<len;i++){
			if(arr[i]>base){
				right.push(arr[i])
			}else {
				left.push(arr[i])
			}
		}

		// 递归分割
		// 递归合并
		return quick(left).concat(base,quick(right));
	}

	return quick(arr);
}

```


### 六，数组自带排序方法
##### 原理：
1. JavaScript 数组自带的排序方法，功能更丰富.
2. 数组自带的排序方法，引擎内部会优化，跟快速排序性能差不多。

##### 实现：
```javascript
// 数组自带
function arraySort(arr){
	arr.sort(function(a,b){
		return a - b;
	});
}
```



### [源代码](https://github.com/luckybirdme/luckybirdme.github.io/blob/master/example/js/my-sort-func.html)