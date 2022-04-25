---
title: SGX的验证
tags:
  - SGX
categories:
  - 安全
  - 可信执行环境
date: 2020-06-24 17:50:38
---

Intel SGX可以很好的保证enclave（安全区域）内代码和数据的隐私性和完整性。但是如果需要多个enclave合作的话，在合作之前需要确认对方时SGX enclave。这种验证机制并不简单，分为本地验证和远程验证两种，下面会分别介绍。SGX的验证机制有很多细节，这里不会涉及太多的细节，重在帮助大家理解SGX的验证机制，更好的理解SGX的安全性以及它可能存在的弱点。

<!--more-->

## 本地验证 *Local Attestation*

**SGX的本地验证用于一个enclave（称之为被验证enclave）向同一机器上的另一个enclave（称之为目标enclave）证明它的身份。**

本地验证流程：

1. 被验证enclave调用**EREPORT**指令，，生成验证报告并发送给目标enclave
2. 目标enclave收到验证报告后，判断验证报告是否可信

**EREPORT**产生的**验证报告**中包含以下信息：

- **MAC（Message Authentication Code）标签**

  由MAC加密系统产生的标签。

  > MAC加密系统：
  >
  > 发送者利用MAC算法读取对称密钥和一个变长的消息，产生一个定长的MAC标签。接受者只要提供原始消息、对称密钥和MAC标签，就可以验证消息的真实性。

- enclave当前的身份信息

  - enclave的measurement
  - 基于证书的身份信息（比如软件开发商的安全版本号等）

- SGX实现的安全版本号（CPUSVN）

- enclave提供的64-byte（512-bit）的消息

- **KEYID**

  enclave初始化时产生的一个随机数。

验证报告的详细内容如下图：

![image-20200624170332771](https://tva1.sinaimg.cn/large/007S8ZIlly1gg3hm2b6arj30ic10kgof.jpg)

目标enclave想要通过MAC标签验证消息是由SGX实现发出的，那么**生成MAC标签的对称密钥只能SGX实现和目标enclave知道**。被验证enclave是不可以知道这个对称密钥的，**一旦这个对称密钥泄露，验证报告就可能被伪造**。

这个对称密钥成为*Report Key*，由**EGETKEY**指令产生。**EGETKEY**指令生成密钥的依据包括：

- 嵌入到处理器中的一个秘密
- 包括**目标enclave**的measurement在内的一些信息

目标enclave可以自己生成相应的对称密钥，验证报告中的信息。

目标enclave可以确定验证报告中的MAC标签是由SGX实现生成的，原因有二：

1. **EGETKEY**生成密钥的算法和MAC加密系统的MAC算法都是保密的，只有SGX实现才能产生这个MAC标签
2. 只有SGX实现可以读取内嵌在处理器中的秘密

## 远程验证 *Remote Attestation*

相比于本地验证，远程验证要更加复杂。在介绍远程验证前，先介绍将会涉及到的一些内容：

在生产的过程中，处理器中的一个叫做e-fuse存储器中烧有两个秘密，我们称这两个秘密为*Provisioning Secret*和*Seal Secret*。

*Provisioning Secret*是intel的密钥生成工具产生的，处理器生产过程中与intel的密钥生成工具通信，获取*Provisioning Secret*。在烧入处理器e-fuse的同时也会被intel保存在数据库。

*Seal Secret*是在处理器芯片中生成的，所以Intel不知道这个秘密是什么。*Seal Secret*存在的好处在于，即便攻击者攻破了intel的密钥生成工具，也没办法生成**EGETKEY**生成的密钥。*Seal Secret*只有被**EGETKEY**做为密钥生成依据时才可以被访问，而不会暴露给任何软件。

远程验证流程：

1. 被验证enclave利用**EGETKEY**指令生成*Provisioning Key*，生成*Provisioning Key*的依据包括：

   * *Provisioning Secret*
   * enclave当前以证书为基础的身份信息
   * SGX实现的安全版本号

2. 被验证enclave利用*Provisioning Key*向intel的*Provisioning*服务证明自己是可信的。

   上面提过，intel记录了*Provisioning Secret*，同时拥有生成密钥的算法。

3. 通过验证后，intel的*Provisioning*服务会生成一个*Attestation Key*并返还给被验证enclave

4. 被验证enclave再利用**EGETKEY**指令以*Seal Secret*等信息为依据生成*Provisioning Seal key*，用这个密钥加密*Attestation Key*后存入计算机内存或磁盘。

5. *Quoting Enclave*通过**EGETKEY**获得*Provisioning Seal key*，从内存中读取并解密*Attestation Key*

6. 被验证enclave向同一机器上的一个成为*Quoting Enclave*的特殊enclave执行本地验证

7. *Quoting Enclave*将收到的本地验证的验证报告中的MAC标签换成由*Attestation Key*产生的签名（一种intel的特殊签名机制）

8. *Quoting Enclave*将远程验证报告发送给目标enclave



## References

1. Costan, Victor, Ilia Lebedev, and Srinivas Devadas. "Secure processors part I: background, taxonomy for secure enclaves and Intel SGX architecture." *Foundations and Trends in Electronic Design Automation* 11.1-2 (2017): 1-248.
