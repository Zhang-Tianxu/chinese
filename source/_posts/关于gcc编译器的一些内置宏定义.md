---
title: 关于gcc编译器的一些内置宏定义
tags: gcc 编译器 宏定义 __FILE__ __LINE__ __TIMESTAMP__
categories:
  - 学习
  - 计算机及软件
  - 编程语言
  - c/c++
date: 2019-01-23 20:00:29
---

# gcc内置宏

在看一些c语言代码时经常突然冒出一些宏，比如`__FILE__`、`__TIME__`等。前面也没有出现相关的定义，一直都没想着弄清楚是什么。这次又看到了这些宏，花了一点时间看了下是什么。

发现这些其实都是编译器内置的宏定义，所以代码中没有给出定义就能直接使用。这样的宏还不少，列出几个常见的，例如：

```c
#include <stdio.h>

int main()
{
	printf("__FILE__ is %s \n",__FILE__);
	printf("__LINE__ is %d \n",__LINE__);
	printf("__FUNCTION__ is %s \n",__FUNCTION__);
	printf("__DATE__ is %s \n",__DATE__);
	printf("__TIME__ is %s \n",__TIME__);
	printf("__TIMESTAMP__ is %s \n",__TIMESTAMP__);
	return 0;
}
```

输出结果是：

<!--more-->

```c
__FILE__ is D:\桌面\常用文件\学习文档\专业学习\C++项目\未命名1.cpp
__LINE__ is 6
__FUNCTION__ is main
__DATE__ is Jan 23 2019
__TIME__ is 20:07:46
__TIMESTAMP__ is Wed Jan 23 19:56:53 2019

--------------------------------
Process exited after 0.09669 seconds with return value 0
请按任意键继续. . .
```

从上面的代码和输出结果可以看出

* \_\_FILE\_\_
  指本语句所在源文件的名称（绝对路径）。
* \_\_LINE\_\_
  指本语句所在的行号。
* \_\_FUNCTION\_\_
  指本语句所在函数的函数名。
* \_\_DATE\_\_
  指本语句所在源文件**编译时**的日期。
* \_\_TIME\_\_
  指本语句所在源文件**编译时**的时间。
* \_\_TIMESTAMP\_\_
  指本语句所在源文件**创建时**的日期和时间。