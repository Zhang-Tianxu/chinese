---
title: leetcode 2 AddTwoNumbers
tags:
  - leetcode
  - 力扣
  - algorithm
  - 算法
  - medium
  - 中等
categories:
  - 学习
  - 计算机及软件
  - 数据结构与算法
  - leetcode
mathjax: true
date: 2019-02-10 22:11:01
---


# 2. Add Two Numbers

## 问题描述

给定两个**非空**链表来表示两个非负整数。整数的数位反向存储，每个链表节点存储一个数字。将两个整数相加并以相同的格式返回结果。

例如：

>输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
>
>输出：7 -> 0 -> 8
>
>其实就是 342 + 465 = 807

<!--more-->

## 解题思路

这个题目本身难度不大，不知道难度为什么是*中等*。我解题过程中主要的纠结点是对结果链表头部的处理，因为链表头部相对于其他节点比较特殊，需要单独考虑。其他没有发现什么难点，无非是对链表和指针的操作。

## 具体实现

对于链表头节点的问题，我选择创建一个空的头节点，这样可以统一处理头节点和其他节点，在最后返回时跳过空的头节点。

我的第一次实现：

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def addTwoNumbers(self, l1: 'ListNode', l2: 'ListNode') -> 'ListNode':
        carry = 0 #进位
        
        lp1 = l1
        lp2 = l2
        
        lsum = 0
        
        res = ListNode(0)
        lpr = res
        
        if not lp1:
            return l2
        if not lp2:
            return l1
        
        while(lp1 and lp2):
            lsum = lp1.val + lp2.val + carry
            if(lsum >= 10):
                lsum -= 10
                carry = 1
            else:
                carry = 0
            new_digit = ListNode(lsum)
            lpr.next = new_digit
            lpr = lpr.next
            lp1 = lp1.next
            lp2 = lp2.next
        while(lp1):
            lsum = lp1.val + carry
            if lsum >= 10:
                lsum -= 10
                carry = 1
            else:
                carry = 0
            new_digit = ListNode(lsum)
            lpr.next = new_digit
            lpr = lpr.next
            lp1 = lp1.next
        while(lp2):
            lsum = lp2.val + carry
            if lsum >= 10:
                lsum -= 10
                carry = 1
            else:
                carry = 0
            new_digit = ListNode(lsum)
            lpr.next = new_digit
            lpr = lpr.next
            lp2 = lp2.next
        if carry != 0:
            new_digit = ListNode(carry)
            lpr.next = new_digit
        res = res.next
        return res
```

leetcode运行时间128ms。

第一次的实现表现不佳，用时排名接近50%。主要问题在于每次循环都用if/else判断lsum是否大于等于10，这样其实浪费了不少时间。那么我们能不能无论lsum是否小于10，都统一处理呢？可以！

优化后代码：

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def addTwoNumbers(self, l1: 'ListNode', l2: 'ListNode') -> 'ListNode':
        carry = 0 #进位
        
        lp1 = l1
        lp2 = l2
        
        lsum = 0
        
        res = ListNode(0)
        lpr = res
        
        while(lp1 and lp2):
            lsum = lp1.val + lp2.val + carry
            carry = lsum // 10
            lsum = lsum % 10
            new_digit = ListNode(lsum)
            lpr.next = new_digit
            lpr = lpr.next
            lp1 = lp1.next
            lp2 = lp2.next
        while(lp1):
            lsum = lp1.val + carry
            carry = lsum // 10
            lsum = lsum % 10
            new_digit = ListNode(lsum)
            lpr.next = new_digit
            lpr = lpr.next
            lp1 = lp1.next
        while(lp2):
            lsum = lp2.val + carry
            carry = lsum // 10
            lsum = lsum % 10
            new_digit = ListNode(lsum)
            lpr.next = new_digit
            lpr = lpr.next
            lp2 = lp2.next
        if carry != 0:
            new_digit = ListNode(carry)
            lpr.next = new_digit
        return res.next
```

leetcode运行时间96ms。

96ms的表现已经超越99.26%的实现。

## 超慢实现

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        if l1 is None:
            return l2
        if l2 is None:
            return l1
        else:
            l3val = 0
            add1 =0 
            l3 = ListNode(0)
            l3last = l3
            while l1 or l2 :
                l3val=0
                if l1 :
                    l3val = l1.val
                    #print(l1.val)
                    l1 = l1.next
                    
                if l2 :
                    l3val += l2.val
                    #print(l2.val)
                    l2 = l2.next
                l3val += add1
                add1 = l3val//10
                l3last.next = ListNode(l3val%10)
                l3last = l3last.next
                print(add1)
            
            if add1==1:
                l3last.next = ListNode(1)
                
            
            return l3.next    
```

leetcode用时312ms。

这个超慢实现，我不太确定慢的原因。我觉得可能的原因有：

1. 循环中的两个分支多数情况下都要经过，这限制了处理器的预执行优化，影响性能。
2. 实现中有`print()`

## 参考文档

1. Leetcode国际版：https://leetcode.com/
2. 可能漏掉某些参考文档，请作者联系添加引用或删除相关内容。
