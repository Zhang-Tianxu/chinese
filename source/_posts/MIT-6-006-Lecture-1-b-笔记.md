---
title: MIT 6.006 Lecture 1-b 笔记
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
top: -1
date: 2019-01-20 23:15:08
---



# Lecture 1-b 笔记——Peak finder

这节主要讲解”**极值点问题**“（Peak finder），通过不同解决思路之间的对比来理解如何设计高效算法。

![MIT 6.006 Lecture 1-b](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/algorithm/MIT-6-006/MIT6_006_1_b.png)

<!--more-->

> *We pick this problem, peak finder, because it's so easy to understand, and there are fairly straightforward algorithms that are not particularly efficient to solve this problem. And so this is kind of toy problem but like a lot of toy problems, it's very evocative, and that it points out the issue that involve in designing efficient algorithm.*
>
> (*选择“极值点问题”，是因为它很容易理解，而且有很多虽不高效但简单的方法解决它。它就像一个练习问题，像其他练习问题一样具有启发性，会指出设计高效算法需要注意的点。*)

## One-dimensional version

![MIT 6.006 Lecture 1-b-2](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/algorithm/MIT-6-006/MIT_6_006_1_b_2.png)

> Position 2 is a peak if and only if b $\geq$ a and b $\geq$ c
>
> Position 11 is a peak if k $\geq​$ j
>
> (*位置2是极值点当且仅当* b $\geq$ a $\bigwedge$ b $\geq​$ c
>
> 位置11是极值点，只需k $\geq​$ j)

**Problem:** Find **a** peak if it exists.（极值点是否存在（值得思考的问题）？如果存在，找到其中一个。）

### Straightforward Algorithm:

从左向右遍历

![MIT 6.006 Lecture 1-b-3](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/algorithm/MIT-6-006/MIT_6_006_1_b_3.png)

1. 如果位置$\frac{n}{2}​$左边的数从左向右增大，而右边从左向右减小，那位置$\frac{n}{2}​$就是一个极值点。
2. 如果整个数组从左向右一直增大，那位置$n$就是极值点。

&emsp;上述第1中情况，我们遍历$\frac{n}{2}$个元素找到极值点。而第2中情况我们要遍历全部$n​$个元素才能找到极值点。

所以**最坏情况**下的时间复杂度是$\Theta(n)​$。

<font color=red>注意符号$O(n)$、符号$\Theta(n)$和$\Omega(n)$在算法时间复杂度上的意义。</font>

### Divide&Conquer algorithm:

![](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/algorithm/MIT-6-006/MIT_6_006_1_b_4.png)

这次我们从$\frac{n}{2}$位置开始分别和左右两个位置比较。

> If $a[\frac{n}{2}] < a[\frac{n}{2}-1]​$ then only look at left half 1 ... $\frac{n}{2}-1​$ to look for **a** peak.
>
> Else if $a[\frac{n}{2}] < a[\frac{n}{2}+1]$ then only look at right half $\frac{n}{2}+1$ ... $n$ to look for **a** peak.
>
> Otherwise $\frac{n}{2}$ position is a peak.
>
> (*如果*$a[\frac{n}{2}] < a[\frac{n}{2}-1]$，左半边数组中一定存在极值点。
>
> *如果*$a[\frac{n}{2}] < a[\frac{n}{2}+1]$，右半边数组中一定存在极值点。
>
> 如果上述两种情况都不成立，$\frac{n}{2}$就是极值点。)

下面分析这个算法的时间复杂度：

$T(n) = T(\frac{n}{2}) + \Theta(1)$

从上面的时间复杂度分析可以得到

$T(n) = T(\frac{n}{2}) + \Theta(1) $

&emsp;&emsp;$= T(\frac{n}{4}) + \Theta(1) + \Theta(1)$

&emsp;&emsp;$ = \log_{2}n\times \Theta(1)$

&emsp;&emsp;$=\Theta(\log_{2}n)$

### 对比

在Python环境下分别编写上面两个算法解决极值点问题，当$n = 10,000,000$时，线性时间复杂度算法用时约13秒，而对数时间复杂度算法用时仅0.001秒。当输入足够大时，两种时间复杂度的差别还是很大的。

对于一维Peak Finder问题的具体解决，将看[传送门](https://freshmanhaner.github.io/2019/01/23/leetcode-162-Find-Peak-Element/#more)。

这种差别在二维的情况下更为明显：

## Two-dimensional version

![MIT 6.006 Lecture 1-b-5](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/algorithm/MIT-6-006/MIT_6_006_1_b_5.png)

> a is a 2-dimensional peak if and only if $a \geq b,a \geq c,a \geq d,a \geq e​$.
>
> *二维极值点的定义：当且仅当$a \geq b,a \geq c,a \geq d,a \geq e$时 $a$是一个二维极值点。*

**Problem:** Find **a** peak if it exists.（极值点是否存在（值得思考的问题）？如果存在，找到其中一个。）

### Greedy Ascent alorithm:

![MIT 6.006 Lecture 1-b-6](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/algorithm/MIT-6-006/MIT_6_006_1_b_6.png)

需要选择一个开始点，我们假设从二维数组的中间开始向右，当然你也可以选择其他的出发点。

我们从12开始向右，发现右边比12大，所以移动到13，然后是14，到边界转而向下……

（其间如果遇到比当前位置小的，就转向其他方向）

如图，我们一直到20的位置，发现20是一个极值点。不难发现**最坏情况**下的时间复杂度是$\Theta(m\times{n})$。

### Extent 1-D devide&conquer to 2-D(Attempt No.1)

> Pick middle column $j = \frac{m}{2}$
>
> Find a 1-D peak at $(i,j)$
>
> Use $(i,j)$ as a start point on row $i$ to find 1-D peak on row $i$
>
> （挑选中间列$j$
>
> 从$j$列中找出一维极值点$(i,j)$
>
> 再在$(i,j)$所在的$i$行找出一维极值，这个就是我们要找的二维极值。）

如果这个算法是正确的，那么它的时间复杂度是$\Theta(\log_{2}n + \log_{2}m)$。这个算法是高效的，可惜并不正确，因为在第$i$行可能并不存在一个二维极值点。

![MIT 6.006 Lecture 1-b-7](https://my-blog-1256501598.cos.ap-beijing.myqcloud.com/github-page/learn/CS/algorithm/MIT-6-006/MIT_6_006_1_b_7.PNG)

如上图，假设从第3列开始，算法会结束在14这个位置，因为12是第3列的一维极值点，但是第2行中却不存在一个二维极值点。

<center><font color=red>Attempt Failed</font></center>

### Extent 1-D devide&conquer to 2-D(Attempt No.2)

> Pick middle column $j = \frac{m}{2}$
>
> Find global maximum on column $j$ at $(i,j)$
>
> Compare $(i,j-1),(i,j),(i,j+1)​$
>
> Pick left columns if $(i,j-1) > (i,j)​$
>
> Else if$(i,j+1) > (i,j)​$  pick right columns
>
> Otherwise $(i,j)$ is a 2-D peak
>
> If you pick left/right columns, you will solve the new problem with half the number of columns
>
> （挑选中间列$j$，找到$j$列中的全局最大值$(i,j)$。
>
> 比较$(i,j-1),(i,j),(i,j+1)$，如果$(i,j-1) > (i,j)$则去左边列中找二维极值点，
>
> 如果$(i,j+1) > (i,j)$则去右边列中找二维极值点，如果上述两种情况都不成立，$(i,j)$就是一个二维极值点。
>
> 如果是去左（或者右）列中找二维极值点，其实就是把问题的列数缩小到了一般。）

&emsp;在上述的算法中，如果只有1列，我们只需找到这一列中的最大值，这就是二维极值了。

&emsp;下面我们来分析这个算法的时间复杂度：

$T(n,m) = T(n,\frac{m}{2}) + \Theta(n)​$

&emsp;&emsp;$ = \Theta(n) \times \log_{2}m​$

&emsp;&emsp;$ = \Theta(n\log_{2}m)$

<center><font color=red>Attempt Succeed</font></center>

**作业：**

&emsp;分析上述算法，至少证明其中一个算法的正确性，或者找出例子证明不正确性。