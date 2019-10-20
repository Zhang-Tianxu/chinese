---
title: git不能pull和push
date: 2018-09-26 15:41:09
tags:
	- git
	- 工具
categories:
	- 学习
	- 计算机及软件
	- 技巧&工具
---
&emsp;最近使用GitHub时，总是没办法push。尽管ssh -T能够成功，GitHub Desktop上也能够push。
<!--more-->
&emsp;我检查了许多遍，[github连接流程](https://www.cnblogs.com/yunquan/p/4862723.html)都没有问题。以前的使用中都是没有问题的。查了许久，在尝试中把之前流程中的
> git remote add origin git@github.com/freshmanHaner/test.git

改成了
> git remote add origin git@github.com:freshmanHaner/test.git

然后再pull或者push 就能成功了。不知道造成错误的原因是什么。