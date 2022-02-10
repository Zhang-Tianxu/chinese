---
title: leetcode 53 Maximum SubArray
tags:
  - leetcode
  - 领扣
  - algorithm
  - 算法
  - easy
  - 简单
categories:
  - 专业学习
  - 数据结构与算法
  - leetcode
mathjax: true
date: 2019-02-26 18:38:34
---


# leetcode 53 Maximum Subarray

**最大子序列和**问题是用于讲解**分治策略**的一个经典例题，这个例题可以帮助我们很好的理解分治策略。但对于这个问题，分治策略并不是最高效的算法。

## 问题描述

给定一个整数数组 `nums` ，找到**其中一个**具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**示例:**

>输入: [-2,1,-3,4,-1,2,1,-5,4],
>
>输出: 6
>
>解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。

<!--more-->

## 解体思路

### 暴力法

暴力法的思路很简单，就是尝试所有可能的子序列起始点，根据排列组合知识很容易知道一共有$A^2_n = n \times (n - 1)$种可能。遍历所有可能并找出最大和，时间复杂度为$\Theta(n^2)$。

### 分治法

分治策略的思想就是递归的解决一个问题，《算法导论》中给出了分治策略的三个步骤：

1. 分解（Divide）：
   将原问题划分为一些子问题，这些子问题和原问题一样，只不过规模更下。
2. 解决（Conquer）：
   递归地解决这些子问题，如果子问题的规模足够小，就停止递归，直接求解。
3. 合并（Combine）：
   将子问题的解合并为原问题的解。

**最大子序列和**问题的分治策略也可以对应上面三步：

1. 分解：
   将整个数组划分为左右两个子数组，分别求两个子数组的最大子序列和。这样两个子问题和原问题一样，规模变为原来一半。

2. 解决：
   递归地分解数组，知道数组中只有一个元素时，可以直接把这个元素作为最大子序列和，返回给上一层。

3. 合并：
   合并过程是这个问题的**关键**。在分解步骤中你可能已经发现，两个子数组的最大子序列和中较大的那个并不一定就是原问题的最大子序列和。这是因为在划分数组的时候将两个子数组完全隔开了，而最大子序列可能会跨越两个子数组，所以要考虑这种情况。
   所以原问题的最大子序列和一共有**三种**可能的情况：

   1. 完全在左边子数组中
   2. 完全在右边子数组中
   3. 跨越左右两个子数组

   上面两种情况只需要递归的解决就可以，而第3种情况需要仔细考虑，想要找出跨越两个数组的最大子序列和需要遍历整个数组，所以这一过程的时间复杂度是$\Theta(n)$。那么可以得出分治策略解决该问题的运行时间：

$$
T(n) = 
\begin{cases}
\Theta(1)&if\ \ n\ =\ 1 \\
2T(\frac{n}{2}) + \Theta(n)&if\ \ n\  > \ 1
\end{cases}
$$

用**主方法**解此递归式得时间复杂度为$\Theta(nlog_2n)$。

### 一次遍历

这个问题最重要的是理解分治策略，理解了分治策略后我们可以进一步优化时间复杂度。实际上我们可以在一次遍历内找出最大子序列和。

从头开始遍历时，每加入一个元素我们都可以比较加入和不加入两种情况下哪一个和更大，这样就会得到这一步的最大子序列和。然后用一个变量保存全局最大子序列和，就能得到我们想要的值。这其实有点类似动态规划。

## 具体实现

### 暴力法

```python
def sum_p(nums: List[int], s: int, e: int) -> int:
    sum = 0
    for i in range(s,e+1):
       sum += nums[i]
    return sum
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        max_subarray_sum = float('-inf')
        for i in range(len(nums)):
            for j in range(i,len(nums)):
                if sum_p(nums,i,j) > max_subarray_sum:
                    max_subarray_sum = sum_p(nums,i,j)
        return max_subarray_sum
```

leetcode美国版超时。国内版96ms，击败9.45%。

### 分治法

```python
def max_cross_subarray(nums: List[int],s: int, m: int, e: int) -> int:
    # 求解跨越子数组的情况
    left_sum = 0
    left_max_sum = float('-inf')
    print(left_max_sum)
    left_i = left_max_i = m
    while left_i >= s:
        left_sum += nums[left_i]
        if left_sum > left_max_sum:
            left_max_sum = left_sum
            left_max_i = left_i
        left_i -= 1
    right_sum = 0
    right_max_sum = float('-inf')
    right_i = right_max_i = m + 1
    while right_i <= e:
        right_sum += nums[right_i]
        if right_sum > right_max_sum:
            right_max_sum = right_sum
            right_max_i = right_i
        right_i += 1
    return left_max_sum + right_max_sum

def maxSubArray2(nums: List[int],s: int, e:int) -> int:
    if s == e:
        return nums[s]
    m = (s + e) // 2
    maxnum1 = maxSubArray2(nums,s,m)
    maxnum2 = maxSubArray2(nums,m+1,e)
    maxnum3 = max_cross_subarray(nums,s,m,e)
    return max(maxnum1,maxnum2,maxnum3)

class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        return maxSubArray2(nums, 0, len(nums) - 1)
```

leetcode用时：124ms，击败5.41%。

### 一次遍历

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        local_max = nums[0]
        global_max = nums[0]
        for i in range(1,len(nums)):
            local_max = max(local_max + nums[i],nums[i])
            if local_max > global_max:
                global_max = local_max
        return global_max
```

leetcode用时：48ms，击败66.85%。

## 参考文档

1. Leetcode国际版：https://leetcode.com/
2. 可能漏掉某些参考文档，请作者联系添加引用或删除相关内容。
