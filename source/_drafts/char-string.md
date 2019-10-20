---
title: 'char[]&string'
date: 2018-10-29 15:50:40
tags:
	- C/C++
	- 字符串
categories:
	- 学习
	- 计算机及软件
	- 编程语言
	- C/C++
---
&emsp;在c/c++中，char[]和string都是字符串的表示形式。但是在处理方面存在差异。
<!--more--> 
&emsp;头文件<cstring>里集合了对char*的操作函数，包括strlen()、strstr()、strcpy()、strcat() memcpy()等等。另外在计算两个char[]字符串的长度差时，可以直接用(char *a;char *b)a - b，是一个比较有用的技巧
&emsp;<string>和<string.h>里则是对string这种非基本类型的定义和操作。包括size()、insert()、erase()、substring()、replace()等等。