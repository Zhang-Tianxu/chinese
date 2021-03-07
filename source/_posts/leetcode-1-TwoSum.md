---
title: leetcode_1_TwoSum
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
date: 2018-11-15 17:29:39
---

# 1. Two Sum
## 问题描述
Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.  
You may assume that each input would have **exactly** one solution, and you may not use the same element twice.  
<!--more-->
**Example:**

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```
## 解题思路
### Brute Force——暴力法
直接采用两层循环扫描所有元素，找到和为target的两个数，然后返回。  
**时间复杂度：$O(N^2)$，空间复杂度：$O(1)$**  

### Two-pass Hash Table——两遍哈希表法
算法在很多时候就是在时间复杂度与空间复杂度的权衡（trade-off）。暴力法空间复杂度很小，时间复杂度却比较大，显然不适用于对时间要求较高的应用。两遍哈希表法就是牺牲一点空间来换取一些时间。之所以叫两遍哈希表法（Hash Table），是因为要循环（iteration）两遍。第一遍建立哈希表存储每个元素的值和索引（Index）。第二遍循环利用建好的哈希表快速查找（Look Up）每个元素是否存在互补元素（即与之相加和为target的元素），就能在$O(N)$的时间复杂度下找到答案。  
**时间复杂度：$O(N)$，空间复杂度$O(N)$**  
&emsp;注意：哈希表的查找速度接近$O(1)$的时间复杂度，但具有不确定性。如果哈希表构建的不够好，碰撞（Collision）很多的话，查找速度也可能退化(Degenerate)为$O(N)$，但是一般而言，哈希表的摊分时间复杂度（Amortized Time Complexity）可以达到$O(1)$。  

### One-pass Hash Table——一遍哈希表法。
其实我们深入研究一下两遍哈希表法会发现，其实可以一次遍历做完**插入**和**查找**两件事。插入之前先看一下现在的哈希表中有没有待插入值的互补值，如果有，后面就不用继续执行了。既减少了遍历的次数，也减少了遍历的长度。当然，这两点只能影响到系数，并不会真正提升时间复杂度。
**时间复杂度：$O(N)$，空间复杂度：$O(N)$**

## 具体实现
### 暴力法
```c++
// Brute Force
//leetcode表现：用时76ms
vector<int> twoSum(vector<int>& nums, int target) {
         vector<int> result;
        int i,j;
        int length = nums.size();
        for(i = 0;i < length;i++)
        {
            for(j = i + 1;j < length;j++)
            {
                if(nums[i] + nums[j] == target)
                {
                    result.push_back(i);
                    result.push_back(j);
                    return result;
                }
            }
        }
        return result;
    }
```
### 两遍哈希表法
```c++
//Two-pass Hash Table
//Leetcode表现：用时4ms
vector<int> twoSum(vector<int>& nums, int target) {
         vector<int> result;
        unordered_map<int,int> hash_table;
        int len = nums.size();
        unordered_map<int,int>::iterator it;
        int i;
        for(i = 0;i < len;i++)
            hash_table.insert({nums[i],i});
        for(i = 0;i < len;i++)
        {
            it = hash_table.find(target - nums[i]);
            if(it != hash_table.end() && (*it).second != i)
            {
                result.push_back(i);
                result.push_back((*it).second);
                return result;
            }
        }
        return result;
    }
```
### 一遍哈希表法
```
//One-pass
//leetcode表现：用时4ms
vector<int> twoSum(vector<int>& nums, int target) {
         vector<int> result;
        unordered_map<int,int> hash_table;
        int len = nums.size();
        unordered_map<int,int>::iterator it;
        int i;
        for(i = 0;i < len;i++)
        {
            it = hash_table.find(target - nums[i]);
            if(it != hash_table.end())
            {
                result.push_back(i);
                result.push_back((*it).second);
                return result;
            }
            else
            {
                hash_table.insert({nums[i],i});
            }
        }
        return result;
    }
```
## 超慢实现
1. 其一：
```c++
//leetcode表现：用时288ms
vector<int> twoSum(vector<int>& nums, int target) {
        vector<int> answer;
        for (int i = 0; i < nums.size(); ++i) {
            for (int j = 0; j < nums.size(); ++j) {
                if (i != j && (nums[i] + nums[j]) == target) {
                    answer.push_back(i);
                    answer.push_back(j);
                    return answer;
                }   
            }   
        }
    }
```
同样是暴力法解决问题，却花费了上一种暴力法接近**四倍**的时间。其中原由我在啊[如何优化程序性能](https://freshmanhaner.github.io/2018/10/05/%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96%E7%A8%8B%E5%BA%8F%E6%80%A7%E8%83%BD/#more)一文中已经提过，这里简单说一下原因。主要原因是在**循环中出现了低效代码***nums.size()*，每次循环（而且还是两层循环）都要调用这个函数，造成了时间上的浪费。这种浪费是非常可怕的，尤其是对于大型程序来说，这样一个简单算法就有数倍的差别，可想而知对于大量数据或者大型程序将有多么大的差别。如果提前把他赋予一个常量，就可以避免这种浪费。  
你以为这已经差到不能更差了？你错了！
2. 其二：
```c++
//leetcode表现：用时292ms
vector<int> twoSum(vector<int>& nums, int target) {
        vector<int> d;
        for (int i=0;i<nums.size();i++)
        {
            for(int j=0;j<nums.size();j++)
            {
                if (i!=j)
                {
                    if(nums[i]+nums[j]==target)
                    {
                        d.emplace_back(i);
                        d.emplace_back(j);
                        return d;
                    }
                }
            }
        }
    }
```
你能找出这个解决方案比“最差”更差的原因吗？要知道*emplace_back()*是比*push_back()*要高效的。
上一种解决方案能别这一种快4ms，在于上一种解决方案少了一些判断。对于*if（断言1 && 断言2）*这种形式，如果断言1不满足，后面的断言二就不会被判断了。上一种解决方案的一些情况中断言2不会被判断，一定程度上节省了时间，二这个解决方案无论什么情况下都会完成两个断言的判断。

## 参考文档

1. Leetcode国际版：https://leetcode.com/
2. 可能漏掉某些参考文档，请作者联系添加引用或删除相关内容。
