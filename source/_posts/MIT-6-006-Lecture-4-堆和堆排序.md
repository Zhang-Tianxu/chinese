---
title: MIT 6.006 Lecture 4 堆和堆排序
tags:
  - algorithm
  - 算法
  - MIT
categories:
  - 算法
mathjax: true
top: -3
date: 2019-02-27 20:55:30
---


# MIT 6.006 Lecture4 堆和堆排序

由优先级队列来引出堆，然后用堆来实现堆排序。

<!--more-->

## 优先级队列 Priority Queue

优先级队列的维基定义：

> In computer science, a priority queue is an abstract data type which is like a regular queue or stack data structure, but where additionally each element has a "priority" associated with it. In a priority queue, an element with high priority is served before an element with low priority. 
>
> 在计算机科学中，优先级队列是一种类似于普通队列或普通栈的抽象数据结构，但是每个元素都会有一个对应的“优先级”。在优先级队列中，优先级高的元素会被优先服务。

MIT 6.006课程给出的定义：

> A data structure implementing a set S of elements, each associated with a key, supporting the following operations:
>
> * insert(S,x): insert element x into set S
> * max(S): return element of S with largest key
> * extract_max(S): return element of S with largest key and remove it from S
> * increase_key(S,x,k): increase the value of element x's key to new value k.(assumed to be as large as current value)
>
> 优先级队列是一个数据结构，这个数据结构实现一个集合S，集合中的每个元素都对应一个关键词，而且这个数据结构支持下列操作：
>
> * insert(S,x): 把元素x插入到集合S中
> * max(S): 返回集合S中有最大关键字的元素
> * extract_max(S): 返回集合S中有最大关键字的元素，并将其从集合中删除。
> * increase_key(S,x,k): 将x元素的关键词的值提升到k。

**优先级队列经常用堆来实现**。

## 堆 Heaps

## 堆排序 Heapsort
