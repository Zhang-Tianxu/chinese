---
title: leetcode 162 Find Peak Element
tags:
  - leetcode
  - algorithm
  - 算法
categories:
  - 算法
  - leetcode
mathjax: true
date: 2019-01-23 11:04:59
---


# leetcode 162 Find Peak Element

这个题目是MIT 6.006算法课中提到的第一个问题，也就是一维数组的Peak Finder问题。如果感兴趣看一看一下我的[MIT 6.006 Lecture 1-b 笔记](https://freshmanhaner.github.io/2019/01/20/MIT-6-006-Lecture-1-b-%E7%AC%94%E8%AE%B0/)。

在对比不同解题思路的同时，我还对比了不同语言（c、C++、Python）。能够非常明显的看出在**效率方面**：c > C++ > Python；时间复杂度最高的**简单算法用c语言写**的效率也要大于C++写的低时间复杂度的算法，更不用说Python。当然如果看简洁程度，Python还是更优。详细情况请看具体实现。

## 问题描述

> A peak element is an element that is greater than its neighbors.
>
> 峰值元素是指比相邻元素大的元素
>
> Given an input array `nums`, where nums[i] $\neq$ nums[i+1], find a peak element and return its index.
>
> 给定输入数组`nums`，规定nums[i]$\neq$nums[i+1]。从该数组中找到一个峰值元素并返回它的索引值。
>
> The array may contain multiple peaks, in that case return the index to any one of the peaks is fine.
>
> 给定的数组中可能含有多个峰值，只需找到其中任意一个即可。
>
> You may imagine that nums[-1] = nums[n] = $-\infty$.
>
> 你可以假设nums[-1] = nums[n] = $-\infty$。

<!--more-->

## 解体思路

题目中很多的假设可以用来提升我们算法的效率，比如nums[i]$\neq$nums[i+1]、nums[-1] = nums[n] = $-\infty$。

### 线性扫描

从左往右扫描整个数组，找出第一个出现的峰值元素。第一反应是看每个元素的左右两边来判断是否为峰值，其实只需要可能右边邻居，因为左边已经比较过了，一定是小于该元素的。

* 时间复杂度
  最坏情况下（元素自左向右递增）我们要扫描整个数组。所以时间复杂度为$O(n)$。

* 空间复杂度

  只用到了常数额外空间，所以空间复杂度是$O(1)$。

### 递归二分搜索

递归二分搜索算法和下面的迭代二分搜索都属于**分治策略**_devide&conquer_的一种。根据题目我们可以看出，考虑中间元素`mid`，如果`nums[mid] < nums[mid+1]`，那么`mid`元素的右边一定存在峰值元素。因为我们只需找出峰值元素中的任意一个，我们就不再需要考虑`mid`元素的左边一半了，这样问题的规模也就缩小了一半。

* 时间复杂度
  $T(n) = T(\frac{n}{2}) + \Theta(1) = T(\frac{n}{4}) + 2\Theta(1) = ... = log_2(n) \times \Theta(1)=O(log_2(n))$
* 空间复杂度
  递归二分搜索中，每次递归表用都会占用上次一半的额外空间，所以总的额外空间是$log_2(n)$。也就是说空间复杂度也是$O(log_2(n))$。

### 迭代二分搜索

迭代二分搜索和递归二分搜索整体思路一样，**时间复杂度也是相同的**（迭代在常数项上一般优于递归）。差别在于迭代二分搜索不会占用太多的额外空间，所以空间复杂度是$O(1)$。

所以能用迭代的地方尽量不用递归。

## 具体实现

### 线性扫描算法——c语言实现

```c
int findPeakElement(int* nums, int numsSize) {
    int i = 0;
    for(;i < numsSize; i++)
    {
        if(nums[i] > nums[i + 1])
        {
            return i;
        }
    }
    return i-1;
}
```
虽然是$\Theta(n)$的时间复杂度，但是用c语言编写的话用时为**0ms**。

### 递归二分搜索——C++实现

```c++
int search(vector<int>& nums, int s, int e){
        if(s == e)
            return s;
        int mid = (s + e) / 2;
        if(nums[mid] < nums[mid + 1])
            return search(nums,mid+1,e);
        else
            return search(nums,s,mid);
    }
    int findPeakElement(vector<int>& nums) {
        int len = nums.size();
        return search(nums,0,len-1);
    }
```
用时**4ms**

### 迭代二分搜索——Python3实现

```python
def findPeakElement(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        start = 0 
        end = len(nums) - 1
        while start <= end:
            if start == end:
                return start
            mid = (start + end)//2
            if(nums[mid] < nums[mid+ 1]):
                start = mid + 1
            else:
                end = mid
```

单纯按照时间复杂度来比较，迭代二分搜索应该是最快的，但是用Python实现用时却是同样时间复杂度C++实现的8倍，用时**32ms**。

## 超慢实现

通过分析超慢实现，我们可以避免影响算法效率的低级错误。

### c语言超慢实现

```c
int findPeakElement(int* nums, int numsSize) {
    int l = 0;
    int r = numsSize  -1 ;
    
    while (l < r) {
        int m = l + (r - l)/2;
        if (nums[m] <  nums[m+1]) {
            l = m + 1;
        } else {
            r = m;
        }
    }
    return r;
}
```

仔细看这段代码，其实并没有什么低价错误，只不过是在计算中间值m的时候算式稍微复杂些。和最优实现0ms差别也不大，用时4ms。

### C++超慢实现

```c++
int findPeakElement(vector<int>& nums) {
        if (nums.size() == 1)
            return 0;
        if (nums.size() == 2)            
            return nums[0] > nums[1] ? 0 : 1;
    //----------------------------------------------
        int peak = 0;
        for (int i = 0 ; i < nums.size(); i++)
        {            
            if (i-1 >= 0 && i+1 < nums.size() )
            {
                if(nums[i] > nums[i-1] && nums[i] > nums[i+1])
                    return i;
            }
            else if (i+1 == nums.size() && nums[i] > nums[i-1])                
            {
                return i;
            }
                
        }
        return peak;
    }
```

和上段代码不同，这段代码就明显犯了**低级错误：在循环代码中加入了低效率因素。**在`for`循环语句中使用`nums.size()`，使得每次循环都会调用`size()`函数，严重影响算法性能，用时8ms。

至于如何优化算法性能，请见[如何优化程序性能](https://freshmanhaner.github.io/2018/10/05/%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96%E7%A8%8B%E5%BA%8F%E6%80%A7%E8%83%BD/#more)

### Python3超慢实现

```python
def findPeakElement(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        
        if len(nums) == 1:
            return 0
        
        if len(nums) == 2:
            return 0 if nums[0] >= nums[1] else 1
        
        low = 0
        high = len(nums) - 1
        
        while low < high:
            
            mid = low + (high - low) // 2
            
            if mid == 0 and nums[0] >= nums[1]:
                return 0
            if mid == len(nums)-1 and nums[-1] >= nums[-2]:
                return len(nums)-1
            if nums[mid-1] <= nums[mid] and nums[mid] >= nums[mid+1]:
                return mid
            if nums[mid+1] > nums[mid]:
                low = mid + 1
            else:
                high = mid
        
        return low
```

循环中加入了太多的分支，一定程度上拖慢了程序的性能，用时76ms。

## 参考文档

1. Leetcode国际版：https://leetcode.com/
2. 可能漏掉某些参考文档，请作者联系添加引用或删除相关内容。
