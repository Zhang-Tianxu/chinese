---
title: leetcode 56 Merge Intervals
date: 2019-02-13 17:41:27
tags:
  - leetcode
  - 领扣
  - algorithm
  - 算法
  - medium
  - 中等
categories:
  - 学习
  - 计算机及软件
  - 算法
  - leetcode
  - 简单
mathjax: true
---

# leetcode 56 区间合并——Merge Intervals

排序算法有很多应用，区间排序就是一种典型的应用。

## 问题描述

给定一组区间，合并所有重叠的区间。

例子1：

> 输入：[[1,3],[2,6],[8,10],[15,18]]
>
> 输出：[[1,6],[8,10],[15,18]]

例子2：

> 输入：[[1,4],[4,5]]
>
> 输出：[[1,5]]

<!--more-->

## 解体思路

按照区间的起始点对区间进行排序，那么可以合并的区间一定是相邻的。

算法步骤如下：

1. 将给定的一组区间**按照起始点进行排序**。
2. 设置*new_start*和*new_end*两个变量来存储即将插入结果队列的区间起始点和终点，从左往右开始遍历排好序的区间。
3. 如果当前区间和下一个区间有重叠，更新*new_end*，**注意新的值一定要大于当前*new_end***。
4. 如果当前区间和下一个区间没有重叠，将*new_start*和*new_end*作为起始点和终点的区间插入结果。

* 时间复杂度
  对序列排序的时间复杂度是$\Theta(n \times log_2{n})$，而对序列遍历的时间复杂度是$\Theta(n)$。所以整个算法的时间复杂度是$\Theta(n \times log_2{n})$。
* 空间复杂度
  空间复杂度和排序算法有关。比如**空间复杂度**是$\Theta(1)$，而合并排序的时间复杂度是$\Theta(n)$。



## 具体实现

我一开始的实现不够简洁，逻辑虽然简单，但是代码看上去很臃肿。具体代码如下：

```python
# Definition for an interval.
# class Interval:
#     def __init__(self, s=0, e=0):
#         self.start = s
#         self.end = e

class Solution:
    def merge(self, intervals: 'List[Interval]') -> 'List[Interval]':
        sorted_int = sorted(intervals,key=lambda x:x.start) #sort intervals by start
        
        result = []
        
        length = len(sorted_int)
        if length <= 0:
            return result
        
        
        new_start = sorted_int[0].start
        new_end = sorted_int[0].end
        
        
        
        i = 1
        while(i < length):
            if new_end >= sorted_int[i].start:
                new_end = max(sorted_int[i].end,new_end)
                i = i+1
            else:
                tmp = Interval(new_start,new_end)
                result.append(tmp)
                new_start = sorted_int[i].start
                new_end = sorted_int[i].end
                
        tmp = Interval(new_start,new_end)//处理最后一个区间
        result.append(tmp)
            
        return result
```

**leetcode用时：60ms，排名85.63%。**

在看过大神的代码后学到了一个**技巧**，就是对**访问列表是对负数的应用**。例如`list[-1]`表示访问列表的最后一个元素，这个技巧在这个题目中可以使得代码变得简洁。

```python
# Definition for an interval.
# class Interval:
#     def __init__(self, s=0, e=0):
#         self.start = s
#         self.end = e

class Solution:
    def merge(self, intervals: 'List[Interval]') -> 'List[Interval]':
        if not intervals:
            return []
        sorted_int = sorted(intervals,key=lambda x:x.start) #sort intervals by start
        
        result = [sorted_int[0]]
        for i in range(1,len(sorted_int)):
            if result[-1].end >= sorted_int[i].start:
                if(result[-1].end < sorted_int[i].end):
                    result[-1].end = sorted_int[i].end
            else:
                result.append(sorted_int[i])
                
        return result
```

**leetcode用时：56ms，排名99.24%。**

这个实现相比于第一个实现不仅变得简洁，而且**省去了新建区间的过程**，直接利用已有区间，这样省下了一定的时间，时间性能排名上升。


## 超慢实现

```python
# Definition for an interval.
# class Interval:
#     def __init__(self, s=0, e=0):
#         self.start = s
#         self.end = e

class Solution:
    def merge(self, intervals):
        """
        :type intervals: List[Interval]
        :rtype: List[Interval]
        """
        res = []
        if not intervals:
            return []

        intervals.sort(key=lambda i: i.start)
        temp = intervals[0]
        i = 1
        while i < len(intervals[:]):#在循环中使用len()函数，非常影响性能
            next = intervals[i]
            
            if temp.end < next.start:
                res.append(temp)
                temp = next
            elif temp.end >= next.start:
                temp = Interval(temp.start, max(temp.end, next.end))
            i += 1
        res.append(temp)

        return res
```

**leetcode用时：748ms**

这个超慢实现还是常见的**低效循环**错误，课件只要避免在[如何优化程序性能](https://freshmanhaner.github.io/2018/10/05/%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96%E7%A8%8B%E5%BA%8F%E6%80%A7%E8%83%BD/#more)提到过的常见错误，可以很大程度上提升程序的时间性能。

## 参考文档

1. Leetcode国际版：https://leetcode.com/
2. 可能漏掉某些参考文档，请作者联系添加引用或删除相关内容。