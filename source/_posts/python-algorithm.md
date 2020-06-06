---
title: python 排序算法
date: 2020-05-04 07:54:53
tags: algorithm
---

> 基本算法实现

<!-- more -->


## 冒泡

```python
def bubbleSort(arr):
	length = len(arr)
	# 一共要冒泡 n-1 次
	for i in range(length-1):
		# 从第一个元素冒泡，在倒数第一位产生最大值
		# 从第二个元素冒泡，在倒数第二位产生次大值
		for j in range(length-1-i):
			# 如果较大，则往后冒泡
			if arr[j]>arr[j+1]:
				# 交换两个值
				arr[j],arr[j+1]=arr[j+1],arr[j]
	return arr

arr=[2,4,1,6,3,5,8,9]
print(bubbleSort(arr))

```

## 选择

```python

def selectSort(arr):
	length = len(arr)
	# 一共要遍历 n-1 次
	for i in range(length-1):
		# 默认当前是最小值
		minIndex=i
		# 往后面寻找最小值，然后放到最前面
		for j in range(i+1,length-1):
			# 如果比默认最小值还小，则是更小的
			if arr[minIndex]>arr[j]:
				minIndex=j
		# 如果存在更小值，则交换到前面
		if minIndex!=i:
			arr[i],arr[minIndex]=arr[minIndex],arr[i]
	return arr

arr=[2,4,1,6,3,5,8,9]
print(selectSort(arr))
```

## 插入

```python

def insertSort(arr):
	length = len(arr)
	# 从第二元素开始，跟前面的元素对比
	for i in range(1,length):
		# 默认插入到当前位置
		insIndex = i
		# 保存当前元素，以便后续插入
		insTemp=arr[insIndex]
		# 遍历前面的元素，如果比当前元素大，则往后移一位
		while insIndex>0 and arr[insIndex-1]>insTemp:
			arr[insIndex]=arr[insIndex-1]
			insIndex=insIndex-1
		# 插入到具体位置
		if insIndex!=i:
		   arr[insIndex]=insTemp
	return arr

arr=[2,4,1,6,3,5,8,9]
print(insertSort(arr))

```

## 希尔

```python

def shellSort(arr):
	length = len(arr)
	# 步长采用二分法
	gap = length>>1
	while gap>0:
		# 插入排序
		for i in range(gap,length):
			# 默认插入到当前位置
			insIndex = i
			# 保存当前元素，以便后续插入
			insTemp=arr[insIndex]
			# 遍历前面的元素，如果比当前元素大，则往后移一个 gap
			while insIndex>0 and arr[insIndex-gap]>insTemp:
				arr[insIndex]=arr[insIndex-gap]
				insIndex=insIndex-gap
			# 插入到具体位置
			if insIndex!=i:
			   arr[insIndex]=insTemp
		# 重新修改步长
		gap=gap>>1
	return arr

arr=[2,4,1,6,3,5,8,9]
print(shellSort(arr))
```

## 归并

```python

def mergeSort(arr):
    length = len(arr)
    # 分割到只剩一个元素时返回
    if length < 2:
        return arr

    # 二分法分割数组
    minIndex = length>>1
    # 继续分组，直到单个元素，此时两个数组进行比较，就是相当于两个元素比较
    leftArr=mergeSort(arr[:minIndex])
    rightArr=mergeSort(arr[minIndex:])
    
    # 将要返回的数组
    result=[]

    # 两个数组进行合并
    # 如果数组第一个元素比较小，则先放到结果中，达到排序的效果
    # 以此类推，直到其中一个数组没有元素了
    while len(leftArr)>0 and len(rightArr):
        if leftArr[0]<rightArr[0]:
            # 弹出第一个元素，并且放到结果数组中
            result.append(leftArr.pop(0))
        else:
            result.append(rightArr.pop(0))
    # 如果数组还有多余的元素，则直接追加到结果中
    while len(leftArr)>0:
        result.append(leftArr.pop(0))
    while len(rightArr)>0:
        result.append(rightArr.pop(0))
    return result

arr=[2,4,7,1,6,3,5,8,9]
print(mergeSort(arr))

```

## 快速

```python


def quickSort(arr):
    length = len(arr)
    # 递归出口
    if length < 2:
        return arr

    # 默认取第一个元素为基准
    pivot = arr[0]
    # 小于基准的放左边数组
    leftArr = []
    # 大于基准的放右边数组
    rightArr = []

    for i in range(1,length):
        if arr[i]<pivot:
            leftArr.append(arr[i])
        else:
            rightArr.append(arr[i])

    # 递归按照不同基准分割数组，直到单个元素
    # 此时比较两个数组的同位元素，相当于比较两个元素大小，达到排序效果
    # 合并所有数组，就是有序的数组
    return quickSort(leftArr)+[pivot]+quickSort(rightArr)


arr=[2,4,7,1,6,3,5,8,9]
print(quickSort(arr))



# 上面的方法会重复创建数组 leftArr 和 rightArr, 导致内存浪费
# 通过指针方式，在原始数组操作，可以节省内存
def quickSortBetter(arr):
    # 交换元素
    def swap(arr,i,j):
        arr[i],arr[j]=arr[j],arr[i]
    # 按照基准值分割数组
    def partition(arr,start,end):
        # 如果开始和结束相等，说明只剩一个元素了，直接返回
        if start>=end:
            return
        # 以第一个元素为基准
        pivot=arr[start]
        # 左指针，向右边移动，寻找比基准大的值，然后跟右边交换
        left = start
        # 右指针，往左边移动，寻找比基准小的值，然后跟左边交换
        right = end

        print(pivot,start,end)
        print(arr)

        while left<right:

            # 从左边往右边找第一个比基准大的值
            while left<right and arr[left]<=pivot:
                left=left+1

            # 从右边往左边找第一个比基准小的值
            while left<right and arr[right]>pivot:
                right=right-1
            # 交换这两个值
            swap(arr,left,right)
            print(left,arr[left],right,arr[right])

        print(arr)

        # 将基准值换到中间位置，因为 left 指针是先判断并且 +1 的，所以这里需要 -1
        swap(arr,left-1,start)

        print(arr)

        # 将基准两侧的数组递归分割，去掉当前基准这个元素，即下标为 left-1 的元素
        partition(arr,start,left-2)
        partition(arr,left,end)

    partition(arr,0,len(arr)-1)
    return arr


arr=[2,4,7,1,6,3,5,8,9]
print(quickSortBetter(arr))

```

## 堆

```python


import math
def heapSort(arr):
    # 交换元素
    def swap(arr,i,j):
        arr[i],arr[j]=arr[j],arr[i]
    # 创建大顶堆
    def heapify(arr,parent_index,end_index):
        left_index = 2*parent_index+1
        right_index = 2*parent_index+2

        # 如果超过了数组边界，退出
        if left_index>=end_index:
            return
        
        # 默认左节点跟父节点替换
        replace_index = left_index
        # 如果右节点存在，而且比左节点大，则取右节点
        if right_index<end_index and arr[right_index]>arr[left_index]:
            replace_index = right_index

        # 如果子节点比父节点大，则进行交换
        if arr[replace_index]>arr[parent_index]:
            swap(arr,replace_index,parent_index)

            # 遍历交换节点下面的子节点，以便建立大顶堆
            heapify(arr,replace_index,end_index)

    # 数组长度
    length = len(arr)
    # 初始化大顶堆，将最大值放到顶部
    # 循环遍历每个父节点，P=2*C+1, C=(P-1)/2, 向下取整
    parent_last = math.floor((length-1)/2)
    while parent_last >= 0:
        heapify(arr,parent_last,length)
        parent_last = parent_last - 1

    # 将大顶堆的值和数组最后一个值交换
    # 再次建立大顶堆，将次大值放到顶部
    # 再次交换，将次大值放到数组倒数第二个位置
    # 以此类推，当交换到第一个元素时，数组就排序好了
    last_index = length - 1
    while last_index >= 0:
        swap(arr,0,last_index)
        heapify(arr,0,last_index)
        last_index = last_index - 1
    return arr


arr=[2,4,7,1,6,3,5,8,9]
print(heapSort(arr))

```