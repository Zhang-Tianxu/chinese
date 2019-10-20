---
title: GNU利用关键字__attribute__设置变量属性
tags: 
	- c语言
	- C++
	- 字节对齐
	- __attribute__
categories:
	- 学习
	- 计算机及软件
	- 编程语言
	- c/c++
date: 2019-01-23 19:42:39
---


# GNU利用关键字__attribute__设置变量的属性

关键字`__attribute__`可以用来声明变量、函数参数、结构体、共用体或者C++中类成员的特殊属性。使用方法是在关键字`__attribute__`后跟着用两个小括号包起来的属性声明。例如：

```c
struct a {
	int a1;
	int a2;
} __attribute__((packed));
```

这个例子中声明结构体`a`的特殊属性——字节对齐，`((packed))`这个属性声明是指最小的对齐方式，即1Byte对齐。

```c
#define PGSIZE 1024
struct a {
	int a1;
	int a2;
} __attribute__((__aligned__(PGSIZE)));
```

这个例子也是字节对齐，但是这次是我们自定义的对其方式：按照1024Byte进行字节对齐。

<!--more-->

关于属性的详细介绍请参考：[参考资料——GNU官方文档](https://gcc.gnu.org/onlinedocs/gcc/Attribute-Syntax.html)