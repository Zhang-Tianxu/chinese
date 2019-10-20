---
title: leetcode 7 ReverseInteger
tags:
  - leetcode
  - 领扣
  - algorithm
  - 算法
  - easy
  - 简单
categories:
  - 学习
  - 计算机及软件
  - 算法
  - leetcode
  - 简单
mathjax: true
date: 2018-11-19 15:47:02
top: 5007
---

# Reverse Integer
## 问题描述
Given a 32-bit signed integer, reverse digits of an integer.
<!--more-->
```
Example 1:
Input: 123
Output: 321  

Example 2:
Input: -123
Output: -321  

Example 3:
Input: 120
Output: 21  
```
**Note:**
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range:$[-2^{31},2^{31}-1]$ . For the purpose of this problem, assume that your function *returns 0 when the reversed integer overflows*.

## 解题思路
这个题目比较简单，也没有太多发挥的空间，主要的性能差异产生于实现的细节上。  
大部分人的思路思路都是利用求余的方法从右往左取出x中的数字，然后在将他们组成结果。当然要记得处理超出范围的情况。  
那么同样的思路在实现时可以产生许多的差异。比如：
* 如何处理x的正负情况
  * `if/else`分别处理两种情况
  * 利用编程语言的特性同时处理两种情况
* 如何存储提取出的数字
  * 存储在**队列**中
  * 边提取边处理，不做存储
* 如何判断超出范围的情况
  * 在溢出前做处理
  * 用范围更大的数据类型存储结果，最后再判断是否超出范围

这些细节对结果不会产生太大的影响，但是当大家的性能非常接近时，每1ms的提升都可以超越许多人。  
## 具体实现
我选择了大多数人能实现的，较快的实现。
```c++
//leetcode表现：用时12ms
int reverse(int x) {
        int result = 0;
        while(x != 0)
        {
            if(INT_MAX / 10 < result || INT_MIN/10 > result)
                return 0;
            result = result*10 + x%10;
            x /= 10;
        }
        return result;
    }
```
## 超慢实现
虽然这一题大家的实现在效率差不许多，我们还是来试着分析一下最慢的实现是怎么造成的。
```c++
//leetcode表现：用时32ms
int reverse(int x) {
        long long res = 0;
        while(x != 0){
            res = res*10 + x%10;
            x /= 10;
        }
        if( (res>=INT_MIN) && (res<=INT_MAX)) return res;
        else return 0;
    }
```
这个较慢实现看上去和较好的实现差别不大，唯二的差别是：
* 较慢实现采用计算完成后再判断是否溢出
* 结果的数据类型一个是int一个是long long

上述两点差别在这个题目中并不会造成多大影响。如果非要区别的话，第一个区别在溢出的情况下会多花时间；第二个区别则会造成一点空间上的浪费。

## 参考文档

1. Leetcode国际版：https://leetcode.com/
2. 可能漏掉某些参考文档，请作者联系添加引用或删除相关内容。
