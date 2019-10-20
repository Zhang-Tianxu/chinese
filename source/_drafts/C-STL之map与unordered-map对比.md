---
title: C++ STL之map与unordered_map对比
tags:
  - C++
  - STL
  - 标准模板库
  - map
  - unordered_map
categories:
  - 学习
  - 计算机及软件
  - 编程语言
  - C/C++
date: 2018-11-15 17:54:26
---


map和unordered_map都是STL中的容器，它们虽然用法相似，但是背后的原理值得了解，进而在不同场景中有针对性的应用它们。
<!--more-->
# STL之map
&emsp;[map](http://www.cplusplus.com/reference/map/map/)是C++标准模板库（*Standard Template Library*）中常用的关联容器（*associative container*），元素由键值（*key value*）和映射值（*mapped value*）组成，通过**唯一键值**来查找映射值。map中可以通过键值直接得到映射值，例如：  
```c++
#include <map>
#include <iostream>
#include <string>

int main() {
    std::map<std::string,std::string> name2description;
    name2description["Tom"]="A dumb cat";
    name2description["Jerry"]="A cunning mouse";
    std::cout << "Tom is " << name2description["Tom"] << std::endl;
    return 0;
}
```
# STL之unordered_map
&emsp;[unordered_map](http://www.cplusplus.com/reference/unordered_map/unordered_map/)也是C++标准模板库中的容器，但是不像map那样常用。unordered_map的基本用法和map几乎完全一样，只是不存在反向迭代器（*iterator*）。  

# map与unordered_map的区别
&emsp;虽然两者在使用上几乎没什么区别，但是在用途上却有不小的差别。原因在于__map的内部实现是二叉搜索树（*Binary Search Tree*）__，而且是按键值有序的（可以自定义排序方法）。相对的__unordered_map的内部实现是哈希表__，而且是无序的。它们的内部实现让它们具有了各自的优缺点。  
&emsp;我们知道二叉搜索树各种操作上的性能比较平均也可以接受。不管是查找的O（log N），插入的O（N）还是删除的O（N），虽然都不是最好的情况，但也都是不错的时间复杂度，再加上它是有序的。这使得map容器很常用。  
&emsp;相比于BST的平均，哈希表则更擅长搜索，搜索的摊分时间复杂度达到O（1）。但是它的无序性影响了他的通用性。