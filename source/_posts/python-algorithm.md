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
def quickSort(lists):
    def partition(start,end):
        if start >= end:
            return False
        # 第一个元素为基准
        pivot = lists[start]
        # 低位区间起点
        left = start
        # 高位区间终点
        right = end
        while left < right:

            # 从高位往中间寻找小于基准的元素，移动到低位
            # 首次循环将会移动到基准所在位置
            # 第二次循环起，将会移到低位区间找到的大于基准的元素的位置
            # 由于 right 位置元素可能是上次循环从低位区间转移过来的，所以需要继续判断
            while left < right and lists[right] >= pivot:
                right -= 1
            lists[left]=lists[right]

            # 由于将 lists[right] 更小值移动到 lists[left]
            # 此时 right 下标所在位置元素其实是跟 left 下标值重复

            # 从低位往中间寻找大于基准的元素，移动到高位，即刚才高位区间找到的小于基准的元素位置
            while left < right and lists[left] <=pivot:
                left += 1
            # 由上面可知，right 下标为高位区间的重复元素，刚好可以把大于基准的元素放到此位置上
            lists[right]=lists[left]

            # 由于将 lists[left] 更大值移动到 lists[right]
            # 此时 left 下标所在位置元素其实是跟 right 下标值重复

        # 退出循环条件是 left=right
        # 低位区间 [start, left-1] 小于或等于基准
        # 高位区间 [left+1, end] 大于或等于基准
        # left 所在位置的元素，肯定是重复的元素，所以把基准放到这个位置
        lists[left] = pivot
        # 此时基准已经排好序了，剔除基准后，继续递归低位和高位区间
        partition(start,left-1)
        partition(left+1,end)
    print('test')
    print(lists)
    partition(0,len(lists)-1)
    print(lists)
quickSort([1,2,3,4,5])
quickSort([5,4,3,2,1])
quickSort([5,2,2,1,5])

```

## 堆

```python

def heapSort(lists):
    # 建立大顶堆
    def heapify(root,end):
        # 左节点
        left=root*2+1
        right=left+1
        # 不能超出数组下标
        if left > end:
            return False
        # 默认左节点较大
        max_node=left
        # 如果右节点存在，并且比左节点还大
        if right <= end and lists[right] > lists[left]:
            max_node=right
        # 如果子节点比父节点大，则进行交换
        if lists[max_node]>lists[root]:
            lists[max_node],lists[root]=lists[root],lists[max_node]
            # 递归刚交换过的节点，以便确保二叉树是大顶推
            heapify(max_node,end)
    
    def sort():
        # 遍历每个父节点，即可建立大顶堆
        # 根据大顶堆的子节点和父节点关系，2*parent+1=child
        # 最后一个父节点是 
        list_len=len(lists)
        last_root = (list_len-1)>>1
        while last_root >=0:
            heapify(last_root,list_len-1)
            last_root-=1
        # 将大顶堆的顶部元素，即数组第一个元素，也是最大值，跟最后一位元素交换
        # 除了最后一个元素外，将剩余数组元素继续建立大顶堆，
        # 重复上述两个步骤，直到数组最后一个元素，此时的数组就排好序了
        last_index = list_len-1
        while last_index >=0:
            lists[0],lists[last_index]=lists[last_index],lists[0]
            # 剔除最后一个元素
            heapify(0,last_index-1)
            last_index-=1

    print('test')
    print(lists)
    sort()
    print(lists)

heapSort([1,2,3,4,5])
heapSort([5,4,3,2,1])
heapSort([5,2,2,1,5])

```