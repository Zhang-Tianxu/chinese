---
title: MIT 6.006 Lecture3 插入排序和归并排序
tags:
  - algorithm
  - 算法
  - MIT
  - 名校课程
categories:
  - 学习
  - 计算机及软件
  - 算法
  - MIT 6.006
mathjax: true
top: -2
date: 2019-02-22 11:33:58
---


# MIT 6.006 Lecture3 插入排序和归并排序

这是排序部分的第一讲，我们会**先**介绍一下排序问题及其应用，**然后**介绍**插入排序**和**归并排序**两种算法，并对比。**最后**用Python实现这两种算法。

![MIT 6.006 Lecture3](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/algorithm/MIT-6-006/MIT_6_006_lecture3_1.png)

<!--more-->

## 排序问题及其应用

### 什么是排序问题

>**Input:** 
>	array A[1…n] of numbers.
>**Output:** 
>	permutation B[1…n] of A such that B[1] ≤ B[2] ≤ … ≤ B[n] .
>**e.g.** 
>	A = [7, 2, 5, 5, 9.6] → B = [2, 5, 5, 7, 9.6] 

排序问题给定一个输入数组A，算法处理后输入一个有序数组B。

### 排序算法的应用

排序算法在很多方面都有应用，比如：

* 组织一些数据库，比如歌曲库、书单库、电话簿
* 让某些问题变得简单
  比如找数组中值、二叉查找、找统计离群点等
* 一些不太明显的应用，比如数据压缩、电脑绘图等。

可以说排序算法是计算机科学一个很重要的基础，所以掌握好排序算法是很重要也很必要的。

## 两种排序算法

### 插入排序

插入排序可以说是最简单的一种排序算法，只需要四五行的Python代码就能实现。当然简单在很多时候意味着低效。

伪代码：

```pseudocode
1. Insertion-Sort(A[],n)
2. 	for j <-- 2 to n
3. 		将A[j]插入到已经排好序的子数组A[1..j-1]中
4. 		通过调换位置讲A[j]放在正确的位置
```

![插入排序图解](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/algorithm/MIT-6-006/insertion_sort.PNG)

上述伪代码中第3行，将A[j]插入到已经排好序的子数组，这个过程中**有一定的优化空间**。因为子数组已经是排好序的，所以我们可以选择二分法插入。这样时间复杂度就变成了$\Theta(n)$变为$\Theta(log_2n)$。

即便是用二分查找法找到合适的位置，但是第4行伪代码中将A[j]插入到合适的位置依然要花费$\Theta(n)$。因为插入的过程是通过交换位置实现的。

所以不难看出插入排序要遍历2..n-1元素，将每个元素插入到合适位置的时间复杂度是$\Theta(n)$。所以插入排序的**整体时间复杂度是$\Theta(n^2)​$。**

### 归并排序

归并排序算法是**分治策略**一种典型的应用。《算法导论》中给出了分治策略的三个步骤：

1. 分解（Divide）：
   将原问题划分为一些子问题，这些子问题和原问题一样，只不过规模更下。
2. 解决（Conquer）：
   递归地解决这些子问题，如果子问题的规模足够小，就停止递归，直接求解。
3. 合并（Combine）：
   将子问题的解合并为原问题的解。

对应到归并排序算法

1. 分解：
   将针对整个数组的排序分为对左右两个子数组的排序。
2. 解决
   递归地解决这些子问题，当子问题的规模只剩一个元素是，不用排序就是有序的了。
3. 合并：
   归并排序算法的关键和难点也是在合并这一过程。合并做到将两个有序的数组原地合并为一个有序的数组。

归并排序算法伪代码如下：

```pseudocode
Merge-Sort A[1..n]
	if n = 1
		done
	else
		Merge-Sort A[1..n/2]
		Merge-Sort A[n/2 + 1 .. n]
		Merge the two sorted sub-arrays
```

**分析：**

可以列出归并排序的递归式：
$$
T(n) = 
\begin{cases}
\Theta(1)\ \ 若\ n \le c\\
2T(\frac{n}{2}) + \Theta(n)
\end{cases}
$$
利用**master theory**容易得出归并排序算法的时间复杂度是$\Theta(nlog_2n)​$

## Python实现

python本身已经内置了排序算法，而且时间复杂度不错。我们自己实现一遍主要是为了理解算法。

### 插入排序

```python
def InsertionSort(nums):
	for i in range(1,len(nums)):
		j = i - 1
		tmp = nums[i]
		while j >= 0:
			if nums[j] > tmp:
				nums[j+1] = nums[j]
				j -= 1
			else:
				break
		nums[j + 1] = tmp
```



### 归并排序

```python
def Merge(nums,s,m,e):
	left = nums[s:m+1]
	right = nums[m+1:e+1]

	i = j = 0
	k = s
	
	while i < len(left) and j < len(right):
		if left[i] < right[j]:
			nums[k] = left [i]
			i += 1
		else:
			nums[k] = right[j]
			j += 1
		k += 1
		
	while i < len(left):
		nums[k] = left[i]
		k += 1
		i += 1
		
	while j < len(right):
		nums[k] = right[j]
		k += 1
		j += 1	
		
		
def MergeSort2(nums,s,e):
	if s != e:
		m = (s + e) // 2
		MergeSort2(nums,s,m)
		MergeSort2(nums,m+1,e)
		Merge(nums,s,m,e)
	
	
def MergeSort(nums):
	MergeSort2(nums,0,len(nums)-1)
```

