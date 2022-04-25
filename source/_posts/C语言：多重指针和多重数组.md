---
title: C语言：多重指针和多重数组
date: 2022-04-25 11:51:07
tags:
  - C语言
  - 指针
categories:
  - 学习
  - 计算机及软件
  - 编程语言
  - C/C++
---

<!--more-->

## 一维数组和指针
我们直接看下面代码：

```c
  int a[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9};
  int *b = &a[0];
  int *c = a;

  for(int i = 0;i < 9;i++) {
    printf("%d ", a[i]);
  }
  printf("\n");

  for(int i = 0;i < 9;i++) {
    printf("%d ", b[i]);
  }
  printf("\n");

  for(int i = 0;i < 9;i++) {
    printf("%d ", c[i]);
  }
  printf("\n");
```

由于“维数组的名字等价于数组首元素的地址”，所以上面代码中a、b、c都是指向数组a[10]的首元素（a[0]）的地址，三者在用法上完全相同：

`a[i] == b[i] == c[i] == *(a + i) == *(b + i) == *(c + i)；`

可以看出`b + i == &b[i]`。

## 多维数组和多重指针

一维数组的这个特点让我误以为，下面代码中a和b在用法上也是等价的。

```c
int a[2][3] = {{1, 2, 3}, {4, 5, 6}};
int **b = a;
```

但其实不是的，他们的类型都不相同，a是int指针，而b是int指针的指针。
如果想让b和a的类型相同，应该是：`int *b = a;`，那么这种情况下可以认为a、b等价了吗？还是不行：`a[i][j] != b[i][j]`

两个值不想等的原因：

对a来说： 二维数组本质上也是一维数组，只不过这个“一维数组”的每个元素，是一个一维数组。a[i]表示二维数组的第i个元素（一维数组），`a[i][j]`表示a[i]这个一维数组的第j个元素。

对b来说：b本质是int指针。 `&b[i] = b + i`，其中`b + i`表示指向b后第i个元素的地址，元素就是指针指向的类型，对b来说是int。`b[i]`表示b后第i个int的值，等价于`a[0][i]`。

```c
b[i] = a[0][i];
a[i][j] = b[i * 2 + j];
a[i][j] = *(b + i * 2 + j);
int (*b)[3] = a; // pointer to array[13] of int 
```

## 多维数组作为函数参数
 

多维数组作为函数参数：

```c
 f(int daytab[2][13]) { ... }

 f(int daytab[][13]) { ... }

 f(int (*daytab)[13]) { ... }
```

上面3中声明方式，在函数内都可以使用`a[i][j]`的方式访问数组元素。

> 注意：
> `f(int *daytab[13]) { ... }`这种声明方式，无法使用`a[i][j]`的方式访问数组元素


 那么f(int *a) { ... }可以吗？严格说是可以的，只不过不能使用a[i][j]的方式访问元素，而应该使用*(a + 13 * i + j)的方式访问元素。

 

 多重指针作为函数参数：

 

 用数组做函数参数的方式，数组列数是固定的，如果想要实现动态分配内存，需要使用多重数组作为函数参数:

```c
 f(int **a) { ... }

 

 void foo(int** array, int row_len, int col_len) {

  for(int i = 0;i < row_len;i++) {

   for(int j = 0;j < col_len;j++) {

    printf("%d ", array[i][j]);

   }

   printf("\n");

  }

 }

 

 int **make2DArray(int m, int n) {

  int **arr = (int **)malloc(m * sizeof(int *));

  for (int r = 0; r < m; r++) {

   arr[r] = (int *)malloc(n * sizeof(int));

  }

  for(int i = 0;i < m;i++) {

   for(int j = 0;j < n;j++) {

    arr[i][j] = i * 2 + j;

   }

  }

  return arr;

 }

 

 int main() {

  int **arr1 = make2DArray(5,2);

  int arr2[5][2] = {{1, 2}, {3, 4}, {5, 6}, {7, 8}, {9, 10}};



  foo(arr1, 5, 2);

  printf("\n");

  foo(arr2, 5, 2);

 }
```
