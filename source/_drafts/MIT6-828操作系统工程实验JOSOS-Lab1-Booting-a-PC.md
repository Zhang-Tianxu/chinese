---
title: MIT6.828操作系统工程实验JOSOS Lab1 Booting a PC
date: 2018-11-29 10:33:36
tags:
	- OS
	- 操作系统
	- Lab
	- 实验
	- MIT
categories:
	- 学习
	- 计算机及软件
	- 计算机系统
	- OS
---
先给出MIT的OS Lab1的网址，详细介绍和相关资源在里面都能找到，开始的配置可能要费些力气。[MIT 6.828 Lab1（没被墙）](https://pdos.csail.mit.edu/6.828/2018/labs/lab1/)  
这个实验要求你有比较多的预备知识，包括
1. 汇编语言——[汇编参考资料（注意intel和AT&T语法的不同）](https://pdos.csail.mit.edu/6.828/2018/readings/pcasm-book.pdf)/[这是AT&T的](http://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html)
2. GDB——[官网](https://www.gnu.org/software/gdb/)

整个实验1要实现的代码不难，但是要理解的细节非常多。编程难度不大，理解起来颇为费力，所以要有耐心，多花些时间来理解，如果遇到实在不能理解的东西，可以参考一些别人的理解。还是不行的话，我知道一个做的很好的[答案](https://github.com/Clann24/jos)，写的很详细，编码也很好，但是不到万不得已还是不要点的好。废话不多说，开始吧！

要了解操作系统，首先要了解操作系统是怎么被载入的，因为操作系统归根到底也是一个软件。从计算机启动到载入操作系统的大致过程如下：  

1. 处理器启动时默认访问特定内存地址，这段地址非易失地储存一些命令，完成一些设备的初始化，然后找到引导设备。
2. 从引导设备中读入第一个block，了解loader的信息。
3. 连续读入block来载入操作系统内核。

接下来是关于实验一我的一些理解：
<!--more-->

先给出地址空间的结构，后面是详细介绍：  

![内存结构示意图](https://web-source-1256501598.cos.ap-shanghai.myqcloud.com/image/%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.PNG)



基于intel 8088处理器的早期个人电脑只能够访问1MB的物理地址，所以地址只能是从0x00000000到0x000FFFFF。但是最多只有640KB可用，从0x000A0000到0x000FFFFF这384KB被硬件保留为特殊用途，**最重要的用途是Basic Input/Output System也就是我们常说的BIOS（基本输入输出系统）**，它占用了从0x000F0000到0x000FFFFF的64KB空间。BIOS的主要作用是基础系统的初始化（包括激活显卡，检查内存条数量等等）和操作系统的载入，后面我们再介绍具体怎么载入操作系统。   

intel 80286处理器的出现打破了1MB的限制，可访问的物理内存增加到了16MB，随后的intel 80386更是直接提升到了4GB。intel为了向下兼容，既让基于早期处理器设计的软件依然可以运行，保留了这1MB的结构。所以我们现在使用的电脑内存中会**有一个“洞”**，就是从0x000F0000到0x000FFFFF这64KB。这个“洞”把内存空间分为“传统内存”和“扩展内存”。   

现在的个人电脑处理器多数已经是64位了，64位处理器理论上支持的物理内存已经达到2^64字节也就是18EB（约180亿GB），当然只是理论上……如果你的电脑是64位处理器，并且内存超过4G的话，那你的内存中就会存在**第二个“洞”**。这是因为在32位的物理空间的最上方的一部分内存被保留，用来映射32位设备。（再次）为了兼容，这部分的地址空间也不能被使用。   
介绍完了内存空间的结构，我们正式通过MIT的实验操作系统*JOS*来看操作系统是如何被载入的。

---
讲解细节之前，先来说明一下我们初步目的。MIT给出的JOS操作系统中，kernal的装载主要是由*boot.S*和*main.c*这两个文件完成的。其中*boot.S*由汇编写成，用来初始化设备，将处理器切换为保护模式，最后跳转到*main.c*。*main.c*主要是用来装载kernal，在装载完成后跳转到kernal，并且把控制权交给kernal。接下来我们开始讲解这个目的在细节上是如何实现的。  
按照**实验文档**里的方法顺利用GDB打开了***qemu 模拟器***上*JOS*的启动程序。然后我来逐条分析每条汇编语言的目的。
起始第一条命令是`ljmp`跳转指令：
```assembly
[f000:fff0]    0xffff0:	ljmp   $0xf000,$0xe05b
```
更详细地分析我们知道
1. 开始时CS(Code Segment)寄存器 = 0xf000，IP(Instruction Pointer)指令指针寄存器=0xfff0
2. 第一条命令位于地址*0x000ffff0*处，这是由段地址（CS:IP）转换得到的。而这正是BIOS ROM最高的16bytes。
3. 它要跳转到地址*0x000fe05b*处。

从上面给出的内存地址空间图，我们可以看出0x000ffff0是BIOS的最后16byte，所以他要跳转到0x000fe05b。然后就能够执行BIOS了，BIOS的工作主要是设置中断表，初始化PCI总线和一些设备，最后寻找寻找引导设备（bootable device）。  

如果一个磁盘是bootable的，那它的第一个扇区称为引导扇区（boot sector），里面放置的就是引导装载程序（boot loader）。

BIOS找到引导设备后，会把引导扇区读入内存0x7c00到0x7dff这部分内存中，然后跳转到0x7c00开始执行引导装载程序。

下面是完成这些操作的汇编分析（注意我贴出的都是运行地址，不是链接地址）。

```assembly
[f000:e05b]    0xfe05b:	cmpl   $0x0,%cs:0x6ac8 #(gdb) print/x *(0xf6ac8) 结果$1 = 0x0
[f000:e062]    0xfe062:	jne    0xfd2e1
[f000:e066]    0xfe066:	xor    %dx,%dx #寄存器dx置零
[f000:e068]    0xfe068:	mov    %dx,%ss #寄存器SS置零
[f000:e06a]    0xfe06a:	mov    $0x7000,%esp #extended stack pointer（扩展栈指针）设置为0x7000
[f000:e070]    0xfe070:	mov    $0xf34c2,%edx
[f000:e076]    0xfe076:	jmp    0xfd15c
[f000:d15c]    0xfd15c:	mov    %eax,%ecx
```
这一段对然可以理解每句汇编的意思，但是看不出它的目的。
```assembly
[f000:d15f]    0xfd15f:	cli  #关中断
[f000:d160]    0xfd160:	cld  #状态标志寄存器（flag）的第10位（方向标志位）置零，设置地址的变化方向
```
如果你看反汇编生成的的*obj/boot/boot.asm*，你会发现这两句的链接地址在0x7c00，正是booter里boot.S开始的地方。这两句的意义也不难理解，整个装载过程肯定不能被中断。
```assembly
[f000:d161]    0xfd161:	mov    $0x8f,%eax #将al寄存器置为0x8f(10001111)
[f000:d167]    0xfd167:	out    %al,$0x70  #讲0x8f写入0x70端口，0x70是变址寄存器端口
[f000:d169]    0xfd169:	in     $0x71,%al  #将端口0x71的内容读入al寄存器，0x71是数据端口
[f000:d16b]    0xfd16b:	in     $0x92,%al  #将端口0x92的内容读入al寄存器，0x92是系统控制端口A
[f000:d16d]    0xfd16d:	or     $0x2,%al   #将al寄存器的第2位（位1）置1
[f000:d16f]    0xfd16f:	out    %al,$0x92  #写回端口0x92
```

这段命令目的在于初始化设备。

端口0x92各个位的意义：
* Bit 0 - Setting to 1 causes a fast reset 
* Bit 1 - 0: disable A20, 1: enable A20
* Bit 2 - Manufacturer defined
* Bit 3 - power on password bytes. 0: accessible, 1: inaccessible
* Bits 4-5 - Manufacturer defined
* Bits 6-7 - 00: HDD activity LED off, 01 or any value is "on"

我们看到0x92端口的第2位（位1）置1表示激活A20，即第21个地址线被使能，A20地址线被激活，会使系统工作进入保护模式。  我打印出al寄存器，发现值就是2，也就是说除了位1，其他未全部为零。
```assembly
[f000:d171]    0xfd171:	lidtw  %cs:0x6ab8 
[f000:d177]    0xfd177:	lgdtw  %cs:0x6a74 
```
lidt指令：加载中断向量表寄存器(IDTR)。这个指令会把从地址0xf6ab8起始的后面6个字节的数据读入到中断向量表寄存器(IDTR)中。中断向量表中存放着中断处理程序的首地址，用来处理不同的中断。  
lgdt指令：加载全局描述符表寄存器 GDT（Global Descriptor Table），在GDT中主要存放段描述符，还有其它描述符，它们都是64-bit长。把从0xf6a74为起始地址处的6个字节的值加载到全局描述符表格寄存器中GDTR中。  
全局描述符表实现保护模式非常重要的一部分，因为在实模式下的段号（段描述符）只有16位，这对于32位以上的处理器来说就不够用了，为了向下兼容段号长度又不能更改，只能用一个表来存储段号，原先16位的段号来查找这些表。
```assembly
[f000:d17d]    0xfd17d:	mov    %cr0,%eax
[f000:d180]    0xfd180:	or     $0x1,%eax
[f000:d184]    0xfd184:	mov    %eax,%cr0 
```
这三条命令的目的很明显是将控制寄存器CR0的第1位（位0）置1，CR0的位0是启用保护（Protection Enable）标志。当设置该位时即开启了保护模式；当复位时即进入实地址模式。这个标志仅开启段级保护，而并没有启用分页机制。若要启用分页机制，那么PE和PG标志都要置位。

第一次打开A20地址线是为了检查可用资源，这次是正式进入保护模式了，然后我们需要装载内核了。
接下来boot.S会保存寄存器，并调用main.c的函数来实现引导扇区以及后续扇区的装载，等装载完成，操作权就交到操作系统手里了。*boot/main.c*最后执行的一条语句是
```assembly
 #((void (*)(void)) (ELFHDR->e_entry))();这次是链接地址
    7d6b:       ff 15 18 00 01 00       call   *0x10018
```
然后我们就进入了操作系统的entrypoint。
关于实验的各个练习及问题，[这个连接](https://github.com/Clann24/jos/)都给出了详细的答案。
