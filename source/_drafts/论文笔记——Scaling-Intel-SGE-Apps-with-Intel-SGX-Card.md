---
title: 论文笔记——Scaling Intel SGX Apps with Intel SGX Card
date: 2019-07-08 20:04:21
tags:
  - paper
  - SGX
  - 论文
  - 笔记
categories:
  - 专业学习
  - 学术
  - 论文阅读
mathjax: true
---
# Scaling Intel SGX Apps with Intel SGX Card

这边笔记完全来自下面这篇论文：

Chakrabarti S, Hoekstra M, Kuvaiskii D, et al. Scaling Intel® Software Guard Extensions Applications with Intel® SGX Card[C]//Proceedings of the 8th International Workshop on Hardware and Architectural Support for Security and Privacy. ACM, 2019: 6.

## 太长不看版

本文介绍了一个Intel 提出的一个新硬件——Intel SGX Card，这个SGX Card可以直接插在普通服务器的PCIe插槽上，有了这个新硬件带来一下好处：

1. 使得普通服务器据有SGX特性
2. 突破SGX的128MB的内存限制，使SGX应用程序得到扩展。但是在维持SGX特性的同时实现这种扩展是要在一定程度上牺牲性能的。

本文演示了在不可信数据中心中利用Intel SGX Card部署Intel SGX应用程序的四种方法。 为了加速Intel SGX应用程序的内存扩展部署，论文作者们开发了一个内存共享库，用于主机和SGX Card之间的快速异步通信，并通过VNF用例突出显示SGX Card的能力。
<!--more-->

## 简介

云计算的出现彻底改变了大型互联网应用部署和扩展的方式。但是很多公司还是不想将IT基础设施迁移到云上，原因有二：

1. 通常意义上的安全考量
2. 对于商业机密数据的保护

Intel SGX技术针对在不可信公共云上计算隐私数据这种情况，提供了一种硬件强制实现的可信执行环境。但是到目前为止Intel SGX还是有两个限制：

1. 只支持**单插槽平台**（*single-socket platforms*）
2. 安全内存被限制在128MB以内

本文介绍了一个硬件——SGX Card，SGX Card的特性如下：

* 有三个独立的CPU，用于处理需要安全保证的任务。
* 多个SGX Card可以通过PCI Express总线连接到服务器主机上，提升服务器的额外CPU数量。

有了额外的SGX Card，云服务提供商（CSP）可以使他们现有的服务平台具有Intel SGX的防护能力。随着隐私数据计算需求的增加，CSP们可以逐步为他们的数据中心增加SGX Card。

本文主要描述了在云环境下如何通过SGX Card在保持既定服务器形式的情况下扩展资源密集型SGX应用。我们将展示如何跨SGX Card节点横向扩展SGX应用程序，同时将对性能的影响降至最低。

SGX Card使得Intel SGX

* 支持双插槽服务器平台（*dual-socket server platforms*），并且可以很容易地和现有数据中心基础架构整合。
* 通过软件支持，使得应用程序可以在SGX Card上的三个$Intel^® Xeon^® E3$处理器横向扩展，以获得更多的安全内存。

本文还有如下贡献：

* 对SGX Card的分析
* SGX Card的内存共享库
* 提出了四种能够高效利用SGX Card的软件架构
* 展示了一个SGX Card的应用案例

## Intel SGX

本问讨论如何提升应用程序的**安全性**和**可扩展性**，前者由Intel SGX保证，后者由SGX Card保证。

### Intel SGX架构

Intel SGX是对Intel CPU的一个ISA（指令集架构）扩展，它提供了提升应用程序机密部分机密性和完整性的能力。尽管应用程序的非机密部分处在不可信内存，但是机密部分被放在SGX的enclave中，enclave是一段对包括操作系统和hypervisor在内的其他任意软件都是不透明的内存空间。出于enclave中的代码可以执行几乎所有的CPU指令，并且可以读取enclave内外的数据。然而enclave外的特权/非特权软件对enclave数据的直接读取都会失败。enclave数据离开CPU芯片时，Intel SGX都会对数据作额外的加密。所以尝试直接读取RAM数据的硬件攻击也是无法成功的。

在硬件层面上引入了一些新的x86指令，用于对enclave做initialize、enter、resume和exit操作。当处于enclave模式时，CPU禁止对kernel的上下文切换。所以想要执行系统调用就必须先离开enclave，执行完系统调用后再重新进入SGX。离开enclave时会刷新TLB（Translation Lookaside Buffer），但是TLB对性能来说是很重要的，所以对于I/O密集型应用（需要经常离开enclave）来说，Intel SGX会有比较高的性能开销。

为了实现enclave数据的私密性和完整性，增加了一个硬件——EPC（Enclave Page Cache）。EPC是物理内存中的一个区域，这块区域不能被对应enclave以外的任何软件存取。所有从CPU发送到EPC的数据都会被CPU-secific的临时密钥加密。将数据从EPC移动到CPU会对数据进行解密并确保其完整性。目前，EPC大小为128M，其中只有大约96MB可以被用户数据使用。这么大的内存对某些应用（比如KMS）是足够的，但是如果enclave的工作量超出EPC的大小，就必须执行代价高昂的分页。在现有版本的Intel SGX上想要达到好的性能，就必须通过限制代码和数据大小来避免分页。

### 建立Intel SGX架构

Intel SGX为应用程序提供了熟悉的构建环境。开发过程中的一个根本不同在于要把程序分为可信与不可信两部分，前者放在enclave中。一些类似于Intel SGX SDK的工具需要手动分割代码，但是可以自动生成一些用于两部分代码通信的粘合代码（glue code）。已经有人提出根据开发者提供的代码注释来分割现有应用的自动分割工具。另外还有一些工具允许直接重用现有程序，只需很少甚至不需要任何修改。所有用户代码都在enclave中执行，只有不受信任的I/O系统调用执行在外面。

应用程序不受信的任部分必须将数据传输到安全区，以便enclave对这些数据执行私密计算。 类似地，在安全区完成计算之后，它必须将结果传递回不受信任的应用程序（然后通过不受信任的I/O将这些结果转发给最终用户）。Intel SGX中enclave和不受信任部分代码之间唯一的通信方式是共享不受信任的应用程序内存（enclave可以不受信任内存中的数据）。将通过不受信任内存进行通信分为**同步通信**和**异步通信**两种。

**同步通信：**

![image-20190707190211447](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190707190211447.png)

在同步模式中（如上图），不被信任的部分想要对私密数据执行私密操作时步骤如下：

1. 它必须把数据放在与enclave共享的内存中
2. 然后进入enclave（也就是所谓的ECALL）
3. 然后读取共享内存中的数据进行相应操作。

同样的，enclave想要执行不可信操作时的过程也是：

1. 必须将该操作所需要的数据放入共享内存
2. 然后离开（exits）enclave
3. 然后该操作被不被信任的部分执行（这就是所谓的OCALL）

**异步通信：**

![image-20190707190234316](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190707190234316.png)

在异步模式下，不被信任的部分和enclave利用共享内存以生产者-消费者的模式独立执行。比如，不被信任的部分可能执行I/O操作并且在共享队列中存储网络包，同时enclave按照自己的节奏接受这些包。

不管是异步模式还是同步模式，不被信任的部分和enclave都可以运行在独立的地址空间。通过RPC和共享内存实现ECALLs和OCALLs

## Intel SGX Card

![image-20190707223636636](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190707223636636.png)

上图是SGX Card的硬件架构，其中PCH指平台路径控制器*Platform Controller Hub*，DIMM指双线存储器模块*Dual In-line Memory ModuleSP*指*Scable Processor*。从图中我们可以看出每个SGX Card有三个Intel Xeon E3 CPU，每个CPU有其自己的本地RAM和完整的软件堆栈，所以每个Card节点运行其自己的操作系统并且通过自己的网络与外界通信。这在单个机架中提供了更多的计算能力，从而增加了云密度。

SGX Card通过非透明的PCIe Bridge与主机（通常是Intel Xeon Scable Processor）。SGX Card中的每个处理器都连接着自己的PCH、Flash、DIMM，单个处理器独立工作并且可以被用作三个独立服务器。在本文中SGX Card每个Card节点（card node）。也就是说每个SGX Card包含三个Card节点，每个Card节点像独立计算机一样处理数据，被自己的操作系统管理。

将三个Card节点放在一个PCIe Card上的优点是在Card和主机之间移动数据时能够提供高吞吐量和低延迟，这是Intel SGX Card的显着特征，我们用这种特征来构建内存共享库并高效地扩展Intel SGX应用程序。

### 从软件视角看SGX Card

![image-20190707230800302](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190707230800302.png)

上图是Intel SGX Card的软件架构，可以看出，SGX Card通过建立在PCIe上的虚拟网络和块I/O与主机进行通信。所以每个Card节点有其独立的网络和文件系统。

主机通过专用的带外接口启动并管理每个Card节点，主机也会配置Card节点的网络。Card节点通过专用虚拟以太网链路与主机连接。Card节点的IP地址由主机或者外部DHCP服务器分配。Card节点的虚拟以太网接口和主机网络之间的虚拟桥接器使得Card节点能够与外部网络进行通信。

### Memory access model

![image-20190708093633782](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190708093633782.png)

从途中可以看出SGX Card的每个CPU都运行在自己的内存空间中。这就意味之每个Card节点都有完全独立于主机和其他Card节点的DRMA内存。

如同之前的图2所示，Card上的CPU和主机上的CPU都连接到同一个PCI主线上。这种拓扑逻辑下CPU可以做到通过DMA对其他CPU物理地址的读写，PCI bridges在两边都会暴露一个DMA窗口，通过这个DMA窗口就能调用远程数据。在软件层面，通过正确的内存映射，运行在其中一个CPU上的应用程序可以存取运行在其他CPU上的应用程序的内存。要注意共享内存区域必须是固定的，以防止其他CPU通过DMA读取内存页时，该内存页的拥有者CPU上的操作系统将该页调出内存。

SGX Card在主机侧和Card侧都会暴露DMA engine，DMA engine在两侧可以存取的内存数量是由PCI配置空间寄存器来配置的。远程内存范围可以映射到相应CPU访问范围的MMIO（Memory-Mapped I/O）地址空间。由于主机和Card节点可以具有不同的存储量，因此主机和Card上的DMA窗口大小可以不同。 将DMA窗口映射到MMIO空间后，该范围将显示为系统物理地址空间的一部分，并且可以映射给任何打算访问远程内存的应用程序。

在Card上，只需要将MMIO内存空间映射给应用程序就足够了。此外，DMA还必须被编程为能够成为两侧物理地址空间的桥梁。需要注意的是，利用DMA窗口从主机到Card的通信和从Card到主机的通信是不同的。

Card上的PCI bridges仅桥接Card节点和主机之间的PCI通信，但不桥接多个Card节点之间的PCI通信。 这样做在保证安全的同时也是的Card节点不能通过共享存储器直接相互通信。 如果需要在Card节点之间共享存储器，则可以以两个或更多个Card节点共享主机上的一系列物理存储器，当然这样做会增加内存访问延迟。

### Cache Coherency

尽管主机和Card节点可以共享内存，但是它们的内存空间还是存在Cache Coherency的问题。因为PCI总线不是Cache Coherent的，所以在其中一个内存空间更改内存，并不会自动在另一个内存空间反映为dirty cache line。而且，因为缺少snooping机制，cache line的更新也不会反映在其他内存空间中。造成这些的主要原因是没有能够用来运行cache coherency协议的额外连接。另外对于相同物理地址的地址映射在不同的地址空间可能不同。

所以，运行在主机上的应用程序和运行在Card节点的应用程序的进程间通信会有额外的性能开销。在需要Coherency时为了保证正确性，CPU每次都必须直接（ 绕过Cache）读写远程内存。

为了减轻这个问题，为远程内存映射选取合适的缓存技术变得非常重要。一个明显但是错误的选择是使用默认的写回策略，因为写回策略不会吧Cache line刷到远端（只能由应用程序明确的指定刷新），这回引发应用程序的一些错误行为。另一种常见选择就是写穿策略。

不幸的是，写穿策略不能提供高效的写性能，因为每次写都将分别在PCI总线上发送。所以，为了正确且高效，如果应用想要写到远程内存，这是必须使用*write-combine*缓存：

![image-20190708141304500](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190708141304500.png)

*write-combine*缓存是Intel架构为了高效PCI总线传输而支持的一类内存缓存，它允许数据暂时存放在被称为WCB（*Write-Combine Buffer*）的缓存区，以burst的模式一起释放，而不是将数据作为单个bits/bytes传输。一旦WCB满了，或者明确的刷新时间发生，就会对远程内存执行一次*combine*写。*Write-Combine*提高了PCI总线的利用率，进而提升了性能。

## Intel SGX应用运行在SGX Card上

本文提出四种在SGX Card上设计安全SGX应用的方法：

1. 对于非资源密集型的单enclave SGX应用程序，可以将整个应用程序放入SGX Card节点。
2. 只将应用程序的可信任部分加载到Card节点，其他部分运行在主机上。
3. 如果是资源密集型应用程序、多enclave SGX应用程序，可以*network scale-out*的方法将应用程序部署在Card节点上，然后通过它们之间的虚拟网络实现通信。
4. 最后主机和Card可以使用共享内存而不是网络I/O来实现通信。

一般来说，现代数据中心的服务器都有一些（超过10个）PCIe插口，所以可以容纳多个SGX Card。不过下面的介绍中我们假设只有一个SGX Card。

下面对上面说的4种方法逐个介绍

### 整个应用程序加载到SGX Card节点

对SGX Card的最简单的使用就是将整个应用程序都运行在Card节点上。因为每个Card节点有它自己的操作系统、文件系统以及网络。这种部署不需要对SGX应用进行修改。

![image-20190708143911796](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190708143911796.png)

此类部署通过与已部署的主机服务器共享空间和电源，解决了云密度问题并很好地适应现有数据中心。 每个主机可以继续执行自己的资源密集型计算，并且Card上的Card节点可以托管多个独立的SGX应用程序。

### 只加载可信任部分

前面提到过SGX应用程序的可信部分和不可信部分不能在同一地址空间执行。这使得应用程序的不可信部分运行在主机CPU，可信部分运行在Card节点这种部署方式成为可能。

![image-20190708144302939](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190708144302939.png)

在这种部署方式下，应用程序的不可信部分运行在高性能的服务器上，另一方面，可信部分可以使用Card处理器的所有内核执行enclave中的计算，也可以向外扩展到同一主机上的其他Card节点。

这种部署方式所需要的人工划分的工作应该是最少的。如果原SGX应用程序的可信部分和非可信部分的通信是同步的，那么只有OCALL/ECALL层必须修改，用RPC替换原来的函数调用。如果通信是异步的，那么在应用程序启动期间，必须实例化主机和Card节点之间的共享内存区域。

### 利用Network Scale-Out

上面两种部署方式的一个主要缺陷就是单enclave SGX应用程序会被Card节点的EPC大小所限制。目前Intel SGX服务器的EPC内存是有限的，超过这个内存限制就会出现代价高昂的换页。

有了SGX Card，应用程序可以扩展到多个enclave上执行，比如计算和数据被分开不晒在Card节点上运行的几个enclave上。因此，一个SGX Card就可以将EPC内存扩大原来的3倍，如果有个更多SGX Card则可以将EPC内存扩展到更大。这种部署方式的另外一个好处就是所有enclave共享主机上的不可信RAM。

然而想要采用这种部署方式，原来的SGX应用程序必须经过修改以支持执行在独立EPC内存区域上的多个enclave。特别的，应用程序需要可信部分与不可信部分之间的高效通信方式：要么通过TCP/IP网络消息实现同步通信，要么通过共享内存实现异步通信。

前者比较简单，每个Card节点运行自己的TCP/IP网络栈，并且有自己独立的IP地址，对应用程序的修改和*只加载可信任部分*部署方式类似，用RPC替代直接的OCALL/ECALL。

![image-20190708151514478](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190708151514478.png)

### 利用内存扩展

应用程序的可信部分和非可信部分也可以通过共享内存通信。在启动时，不可信部分会在不可信RAM中实例化一个共享内存区域，然后告诉所有的enclave。在运行时，应用程序的不可信部分和可信部分都遵从一个应用程序指定协议，安全一致地存储/加载数据

![image-20190708152648689](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190708152648689.png)

虽然我们主要使用内存扩展设计来实现主机和Card之间的共享内存通信，但我们也可以使用主机的共享内存作为每个Card节点几乎无限的不可信存储。 但是，Card节点上的enclave必须小心将其私有数据存储在不受信任的主机RAM中，并对其进行适当加密和完整性保护。
可以在PCIe总线上模拟主机和卡之间的共享内存。 由于Intel SGX Card节点与主机之间的共享内存不具有Cache Coherency，因此我们开发了一个库，可缓解Cache Coherency问题，并为不受信任的部分和enclave之间的通信提供高级别API。

## 其他贡献

### 威胁模型

在本文中我们使用和普通Intel SGX相同的威胁模型，可信基只有enclave以及主机和Card上的CPU。不被信任的包括SGX 应用的不可信部分、其他应用、所有的特权软件、主机和不可信SGX Card上的所有的硬件，主机和Card节点的通信也是不被信任的。

理论上，使用Intel SGX Card的部署具有与普通Intel SGX环境相同的威胁模型。用作通信的PCIe总线必须被看作不可信硬件组件，所以enclave必须保护所有的出入数据，用HMAC（*Hash-based Message Authentication Code*）保证完整性，用加密保证私密性，用版本号防止回滚。在两个Card节点的enclave与enclave通信时也需要同样的保护，因为它们的通信需要通过主机内存。如果通过PCIe发送的数据的pattern必须隐藏，那么就必须采用某种混淆计算来消除pattern。

### enclave的远程验证

SGX应用程序的客户端在向enclave传输私密数据之前必须执行远程证明，以获得对enclave内和Intel SGX平台本身执行的代码的信任。对于SGX Card上的部署，远程验证有一下几个选择：

1. 一种简单的方法就是将Card节点暴露在外部网络中，进而暴露给最终用户。然后，客户端对在三个Card节点上运行的三个enclave进行远程证明。 这种方法的缺点是客户必须知道SGX Card上的所有enclave并执行多轮网络通信。
2. 用户将整个服务器看作一个黑盒。每个用户只需要验证作为信任根的那个enclave。这个enclave在去验证其他enclave。如果有多个SGX Card，可以利用信任链验证所有的enclave。
3. 数据中心可能有一个类似OpenStack Barbican的运行在enclave内的密钥管理器，可以证明数据中心所有SGX Card上的所有enclave。因此，该密钥管理器成为信任和供应的单一根。

## 内存共享库

如同上面提到的，SGX Card允许应用程序通过共享内存实现扩展。大量数据可以在运行在主机上的不可信部分和运行在SGX Card上的可信部分间快速传输。

对于Intel SGX应用程序，尽管不可信部分和可信部分运行在不同的计算平台上，但在Card节点上运行的enclave可以直接与主机上的不可信应用程序通信，而不会离开enclave这种模式能够提供性能上的提升。 应用程序的不受信任部分可以在主机平台上进行扩展，而可信部分可以在多个Card节点上进行扩展以满足性能要求。

### 库的设计
我们设计并且建立了一个内存共享库，它为应用程序提供用来利用PCIe主线远程共享内存的高级别API。

![image-20190708161348110](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190708161348110.png)

这个库有三个级别的API：

1. 使用*Write-Combine*的低级别的共享内存访问原语API
2. 为主机和Card节点通信提供的异步队列API
3. 为主机和Card节点之间任务分配提供的任务队列API

低级别API提供了初始化主机和Card节点上的共享内存区域的功能。 初始化内存区域后，开发人员可以调用低级别API函数来读取和写入远程内存（使用写入组合缓存策略）。

异步队列API提供函数在主机和Card节点之间建立一个SPSC（*Single Producer Single Consumer*）共享队列，并且提供两个函数来实现出列和入列。通常，应用程序开发人员将创建与Card节点一样多的队列，以便主机将对特定Card节点的请求放入相应队列进行排队。

任务队列API提供函数建立SPMC（*Single Producer Multiple Consumer*）队列，其他和异步队列API类似。这当主机将可由任何可用Card节点执行的请求排入队列时，这个API对于任务分发非常有用。

### 库的实现

内存共享库的基础是使用SGX Card PCIe驱动提供的API任意编程DMA引擎的能力，该库利用Sysfs等抽象用户空间接口进行DMA引擎编程。

要调度应用程序请求，库必须与远程端的对应方通信。 该通信发生在现有的虚拟网络接口或基于PCI的通信协议上。

在我们的实现远行中在主机上的不可信应用程序存储一块内存然后告诉SGX Card驱动它打算和指定Card节点共享这块内存。同时Card节点上的enclave也会请求驱动将远程内存映射到它的地址空间。结果，主机上的驱动向Card节点上的驱动提供物理地址（或物理地址范围）。 然后，SGX Card上的驱动通过这些远程物理地址之间的映射对DMA引擎进行编程，并将MMIO范围映射到SGX Card上应用程序的地址空间中。

这个基础通信渠道一旦建立，异步通信原语和任务队列就可用于主机上的不可信应用程序和Card节点上的enclave之间的高级别数据交换。

注意到如果应用程序为内存共享保留4KB的页，那内存共享库可能需要在DAM引擎中创建多个映射，因为物理地址可能是不连续的。在理想状态下，应用程序应该使用2MB到1GB的巨页来保留共享内存。使用巨页可确保连续的物理地址范围。 由于操作系统永远都不会讲巨页换出，所以巨页还具有确保内存永久固定的好处。

### 总体性能

通常情况下CPU和DRAM之间的存取时间大约是100ns，而CPU和PCI寄存器之间的存取时间大约在400ns-500ns之间。远程内存读取因为增加了PCI和DRAM之间的存取，所以会额外增加大约100ns的开销。通常，与本地内存相比访问远程内存时开销可能达到约5倍，因此应用程序的吞吐量会下降。然而，吞吐量的这种下降可以通过并行计算来补偿。 通过在Intel SGX Card上的多个Card节点上使用多个Intel SGX enclave，并将请求分配给这些enclave（使用内存共享），我们可以提高应用程序的整体吞吐量。

## 用例——VNF（Virtual Network Function）

为了展示Intel SGX Card的能力，我们从零开发了可扩展的支持Intel SGX的安全监控VNF。我们的安全监视器检查网络上的所有数据包，并根据远程管理员提供的预定义规则进行转发。

### VNF的架构

我们的VNF运行在enclave中，包含L2/L3网络功能，支持Cuckoo哈希查找表和IPSec，关闭TLS/SSL以实现和远程管理员控制台的安全通信。远程管理员与enclave建立安全的SSL通道，执行远程证明，然后将协议规则和哈希表条目发送到enclave。 初始化后，enclave会连续运行安全监视器VNF，并将统计信息发送回远程管理员。

在将VNF一直到enclave的过程中遇到了一些挑战：

1. 在Intel SGX中ECALL/OCALL的代价高昂，所以我们希望尽量减少进出enclave的次数。
2. EPC换页在当前的Intel SGX实现下代价高昂，所以我们力争尽量减小VNF占用的内存大小。

基于上面的设计要求，我们把VNF切分，运行在两个核上：

![image-20190708190831527](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190708190831527.png)

数据包I/O发生在第一个核生的不可信内存中，而安全监控则在第二个内核的enclave内运行。在第一个核上的不可信部分接受来自NIC的数据包，将指向数据的指针通过RX ring转发给第二个核，然后第二个核上的enclave执行真正的VNF处理所有数据包。第一个核从第二个核接受信息并且将被准许的修改过的数据包通过TX ring发回给NIC。安全监视器VNF永远不会离开enclave，其内存占用空间仅包含带有规则的哈希表，而不包括网络数据包。

这个设计在保证监视器规则隐私性和完整性的同时，性能下降少于10%。（注意我们是要保护远程配置的规则/策略，而不是网络数据包内容。）在我们的实验中，我们不会被SGX限制，而是被NIC 40Gbps的容量限制。为了扩展到40Gbps以上，我们将安全监视器VNF移植到Intel SGX Card上。

### 在Intel SGX上扩展VNF

我们在安全监视器用例中使用了内存扩展。为此，我们修改了VNF的receive/transmit rings以在内存共享库上运行。这使我们能够利用主机CPU上的四个核协助NIC将数据包转发到一个Card节点上的四个核。一旦使用内存共享库提供的API初始化底层生产者/消费者异步队列，我们的安全监视器就可以无缝地工作了。在内存扩展的过程中我们没有遇到具体问题。
我们用五个NIC和五个Card节点评估了我们的原型。为了维持吞吐量，我们在主机CPU上使用了20个内核来处理数据包I/O.在实验中，我们使用Cuckoo哈希表执行5元组查找，在384B大小的TCP/IP数据包上有100K条目;每个enclave使用8个超线程。我们的实验表明完美的可扩展性：我们的基于SGX Card的安全监控器在零数据包丢失的情况下实现了200Gbps的吞吐量，是单个Intel SGX enclave的整整五倍。

![image-20190708195153068](http://pubn1s4ko.bkt.clouddn.com/blog/study/pro/paper/image-20190708195153068.png)

## 总结

在本文中，我们演示了在不受信任的数据中心中使用Intel SGX Card部署Intel SGX应用程序的四种方法。 为了促进Intel SGX应用程序的内存扩展部署，我们开发了一个内存共享库，用于主机和SGX Card之间的快速异步通信，并通过VNF用例突出显示SGX Card的能力。



## 参考文献

1. Chakrabarti S, Hoekstra M, Kuvaiskii D, et al. Scaling Intel® Software Guard Extensions Applications with Intel® SGX Card[C]//Proceedings of the 8th International Workshop on Hardware and Architectural Support for Security and Privacy. ACM, 2019: 6.
2. Ittai Anati, Shay Gueron, Simon Johnson, Vincent Scarlata. Innovative technology for CPU based attestation and sealing. HASP’2013 
3. Victor Costan, Srinivas Devadas. Intel® SGX Explained.IACR Cryptology ePrint Archive, 2016 
4. Meni Orenbach, Pavel Lifshits, Marina Minkin, Mark Silberstein. Eleos: ExitLess OS Services for SGX Enclaves. EuroSys'2017 
5. Sergei Arnautov, Bohdan Trach, Franz Gregor, Thomas Knauth, Andre Martin, Christian Priebe, Joshua Lind, Divya Muthukumaran, Dan O'Keeffe, Mark L. Stillwell, David Goltzsche, David Eyers, Rüdiger Kapitza, Peter Pietzuch, Christof Fetzer. SCONE: secure Linux containers with Intel® SGX. OSDI'2016 
6. https://github.com/intel/linux-sgx. Intel SGX for Linux. Accessed: 2019 
7. Andrew Baumann, Marcus Peinado, Galen Hunt. Shielding applications from an untrusted cloud with Haven. OSDI'2014 
8. Chia-Che Tsai, Mona Vij, Donald Porter. Graphene-SGX: A Practical Library OS for Unmodified Applications on SGX. USENIX ATC’2017 
9. Joshua Lind, Christian Priebe, Divya Muthukumaran, Dan O'Keeffe, Pierre-Louis Aublin, Florian Kelbert, Tobias Reiher, David Goltzsche, David Eyers, Rudiger Kapitza, Christof Fetzer, Peter Pietzuch. Glamdring: Automatic Application Partitioning for Intel SGX. ATC’2017 
10. Ofir Weisse, Valeria Bertacco, Todd Austin. Regaining Lost Cycles with HotCalls: A Fast Interface for SGX Secure Enclaves. ISCA'2017 
11. Dmitrii Kuvaiskii, Somnath Chakrabarti, Mona Vij. Snort Intrusion Detection System with Intel Software Guard Extension (Intel SGX). arXiv:1802.00508, 2018 
12. Bohdan Trach, Alfred Krohmer, Sergei Arnautov, Franz Gregor, Pramod Bhatotia, Christof Fetzer. Slick: Secure Middleboxes using Shielded Execution. arXiv:1709.04226, 2017 
13. Hagit Attiya, Amotz Bar-Noy, Danny Dolev. 1995. Sharing memory robustly in message-passing systems. J. ACM 42, 1 1995 
14. https://asylo.dev. Google Asylo. Accessed: 2019 
15. Wenting Zheng, Ankur Dave, Jethro G. Beekman, Raluca Ada Popa, Joseph E. Gonzalez, Ion Stoica. Opaque: an oblivious and encrypted distributed analytics platform. NSDI'2017 
16. sajin Sasy, Sergey Gorbunov, Christopher Fletcher. ZeroTrace:  Oblivious Memory Primitives from Intel SGX. Cryptology ePrint Archive, 2017 
17. https://redis.io. Redis. Accessed: 2019
18. Felix Schuster, Manuel Costa, Cédric Fournet, Christos Gkantsidis, Marcus Peinado, Gloria Mainar-Ruiz, Mark Russinovich. VC3: Trustworthy Data Analytics in the Cloud Using SGX. SP'2015 
19. Olga Ohrimenko, Felix Schuster, Cédric Fournet, Aastha Mehta, Sebastian Nowozin, Kapil Vaswani, Manuel Costa. Oblivious Multi-Party Machine Learning on Trusted Processors. USENIX Security’2016 
20. Tyler Hunt, Zhiting Zhu, Yuanzhong Xu, Simon Peter, Emmett Witchel. Ryoan: a distributed sandbox for untrusted computation on secret data. OSDI'2016 
21. Seongmin Kim, Juhyeng Han, Jaehyeong Ha, Taesoo Kim, Dongsu Han. Enhancing security and privacy of tor's ecosystem by using trusted execution environments. NSDI'2017 
22. Shweta Shinde, Dat Le Tien, Shruti Tople, Prateek Saxena. Panoply: Low-TCB Linux Applications with SGX Enclaves. NDSS’2017 
23. Ming-Wei Shih, Mohan Kumar, Taesoo Kim, and Ada Gavrilovska. S-NFV: Securing NFV states by using SGX. SDN-NFV Security'2016 
24. David Goltzsche, Signe Rüsch, Manuel Nieke, Sébastien Vaucher, Nico Weichbrodt, Valerio Schiavoni, Pierre-Louis Aublin, Paolo Costa, Christof Fetzer, Pascal Felber, Peter Pietzuch, Rüdiger Kapitza. EndBox: Scalable Middlebox Functions Using Client-Side Trusted Execution. DSN’2018 
25. Michael Coughlin, Eric Keller, Eric Wustrow. Trusted Click: Overcoming Security issues of NFV in the Cloud. SDN- NFVSec'2017 
26. Huayi Duan, Xingliang Yuan, Cong Wang. LightBox: SGX-assisted Secure Network Functions at Near-native Speed. arXiv:1706.06261, 2017 
27. Alexey Gribov, Dhinakaran Vinayagamurthy, Sergey Gorbunov. StealthDB: a Scalable Encrypted Database with Full SQL Query Support. arXiv:1711.02279, 2017 
28. Saba Eskandarian, Matei Zaharia. ObliDB: Oblivious Query Processing using Hardware Enclaves. arXiv:1710.00458, 2018 
29. Christian Priebe, Kapil Vaswani, Manuel Costa. EnclaveDB: A Secure Database using SGX. SP’2018 
30. Rohit Sinha, Mihai Christodorescu. VeritasDB: High Throughput Key-Value Store with Integrity. Cryptology ePrint Archive 2018/251, 2018 
31. https://www.intel.com/content/www/us/en/products/servers/accelerat ors/visual-compute-accelerator-SGX accelerator1585lmv.html. Intel Visual Compute accelerator (Intel SGX Card). Accessed: 2018 
32. https://redis.io/topics/cluster-spec. Redis Cluster Specification. Accessed: 2019 
33. https://github.com/twitter/twemproxy. twitter/twemproxy. Accessed: 2019 
34. Yelick, Bonachea, Chen, Colella, Datta, Duell, Graham, Hargrove, Hilfinger, Husbands, and Iancu. Productivity and performance using partitioned global address space languages. PASCO’2007 
35. Aaftab Munshi, Benedict Gaster, Timothy G. Mattson, and Dan Ginsburg. OpenCL programming guide. Pearson Education, 2011 
36. Marcus Brandenburger, Christian Cachin, Rüdiger Kapitza, Alessandro Sorniotti. Blockchain and Trusted Computing: Problems, Pitfalls, and Solution for Hyperledger Fabric. arXiv:1805.08541, 2018
37. Rolf Neugebauer, Gianni Antichi, José Fernando Zazo, Yury Audzevich, Sergio López-Buedo, Andrew W. Moore. Understanding  PCIe performance for end host networking. SIGCOMM '2018 
38. Somnath Chakrabarti, Brandon Baker, Mona Vij. Intel SGX Enabled Key Manager Service with OpenStack Barbican. ArXiv:1712.07694, 2017
39. Jack Regula. Using non-transparent bridging in PCI Express systems. PLX Technology white paper, 2004 
40. http://www.cpushack.com/tag/knights-corner. CPU of the Day: The 61 Knights of the Intel Xeon Phi. Accessed: 2019 
