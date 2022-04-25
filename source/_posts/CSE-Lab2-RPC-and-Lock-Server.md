---
title: CSE Lab2 RPC and Lock Server
tags:
  - RPC
  - Lock
  - 并发
  - 分布式计算
categories:
  - 计算机系统
date: 2018-11-10 18:16:01
---



从这个实验中我们可以学到：    
1. [远程过程调用（Remote procedure call）——RPC](https://en.wikipedia.org/wiki/Remote_procedure_call)
2. 多线程并发，主要是下面四个函数：
    * _pthread_mutex_lock(&mutex)_&_pthread_mutex_unlock(&mutex)_
    * _pthread_cond_wait(&cond, &mutex)_&_pthread_cond_signal(&cond)_
3. 用上面四个函数实现_acquire(lock_id)_&_release(lock_id)_两个函数，用来实现互斥。

<!--more-->

## 学习材料
* 上面四个函数是在_pthread.h_中实现的，详见[传送门](http://pubs.opengroup.org/onlinepubs/7908799/xsh/pthread.h.html)  
* [lab2: RPC&Lock Server](https://ipads.se.sjtu.edu.cn/courses/cse/labs/Lab-2.html)  
* [代码参考](https://github.com/kururu002/CSE_Lab-MIT)

##  远程过程调用 Remote Procedure Call
&emsp;远程过程调用其实是一种进程间的通讯，它让进程可以像调用本地过程那样调用网路中另一地址空间中的过程。RPC是分布式计算的基础之一，简单易用，但是收到网络的影响。
&emsp;RPC 系统包含一下五个部分：
1. RPC数据和信息的**报文格式标准**
2. 打包(marshal)/解包(unmarshal)工具库
3. RPC编译器
4. 服务端框架
5. 客户端框架
6. 捆绑方法

&emsp;下图描绘了RPC的调用过程：  
![RPC_call](https://upload-images.jianshu.io/upload_images/7143349-a9db3c3c85194c6e.png)    
&emsp;这里我们不讨论RPC的实现细节，知识了解RPC以及它的简单实用。RPC的使用如下
* RPC server使用RPC library创建一个RPC server对象去监听某个端口，并且用reg()函数记录不同的RPC handler。
* RPC client创建一个RPC client对象连接RPC server（利用地址+端口），然后调用RPC server上的远程过程。
* RPC有接口标准规定server和client之间参数的传递。
* server和client之间的数据传输需要经过mashal（发送端）和unmashal（接收端）。

&emsp;因为实验对RPC不要求理解原理，大部分的代码已经实现，只需在extent_client.cc中去调用extent_server.cc中的函数就可以了。注意参数的类型和顺序，因为RPC库并不会检查传入参数的类型，如果传递了错误的类型也会被接受，进而出错。

## Lock Server&Locking
### Lock Server
&emsp;Lock Server主要是由_acquire(lock_id)_&_release(lock_id)_这两个函数组成的，Lock Server可以管理许多个“锁”每个“锁”有唯一的整数id，而且每把”锁“在同一时间只能分配给一个client。“锁”的数量是不限制的，如果传入_acquire(lock_id)_的lock_id之前没有出现过，Locker Server会默认加入管理。_acquire(lock_id)_&_release(lock_id)_两个函数的代码如下：
```c++
std::map<lockid_t,bool> lock_table;//全局变量，用来管理“锁”
pthread_cond_t cond;//条件变量
pthread_mutex_t mutex;//互斥变量

lock_protocol::status
lock_server::acquire(int clt, lock_protocol::lockid_t lid, int &r)
{//clt是client_id
  lock_protocol::status ret = lock_protocol::OK;
        // Your lab2 part2 code goes here
  pthread_mutex_lock(&mutex);//用于对锁操作的互斥
  if(lock_table.find(lid) != lock_table.end())
  { 
    while(lock_table[lid] == true)
    {
      pthread_cond_wait(&cond,&mutex);
      //这个函数需要理解一下，它是带锁wait的吗？它可以带锁wait吗？
    }
    lock_table[lid] = true;
  }
  else
  {
    lock_table.insert(std::pair<lock_protocol::lockid_t,bool>(lid,true));
  }
  pthread_mutex_unlock(&mutex);
  return ret;
}

lock_protocol::status
lock_server::release(int clt, lock_protocol::lockid_t lid, int &r)
{
  lock_protocol::status ret = lock_protocol::OK;
        // Your lab2 part2 code goes here
  pthread_mutex_lock(&mutex);
  if(lock_table.find(lid) != lock_table.end())
  {
    lock_table[lid] = false;
    pthread_cond_signal(&cond);
  }
  else
  {
    pthread_mutex_unlock(&mutex);
    return lock_protocol::NOENT;
  }
  pthread_mutex_unlock(&mutex);//对于互斥变量的操作一定要非常小心，不然容易造成饥饿或死锁。
  return ret;
}
```
### Locking
&emsp;有了_acquire(lock_id)_&_release(lock_id)_这两个函数，我们就可以对yfs_client.cc中的函数（读写磁盘）的函数加锁以实现并行操作了。需要注意的是，有调用关系的函数在加锁是要格外注意。考虑下面两个函数的加锁情况，会出现死锁吗？    
```c++
void isFileExist(int inode_id)
{
    acquire(inode_id);//对inode的加锁，一般直接用inode number做id
    Read(inode_id);
    WriteDisk(inode_id);
    release(inod_id);
}

void readFile(int inode_id)
{
    acquire(inode_id);
    isFileExist(inode_id);
    release(inode_id);
}
```
&emsp;上面的代码明显是不可以的，readFile里已经对inode_id对应的锁加了锁，去查看该文件是否存在时又要这把锁的花，就已经被占用了，isFileExist()会等待readFile()放开这把锁，这在isFileExist()函数返回之前，明显是不可能发生的，这就造成了死锁。    
&emsp;所以**对于需要加锁的函数，应该尽量避免相互调用**。因为这会是的加锁关系相当的复杂，要么不能保证互斥，要么会产生死锁。如果一定要存在调用关系的话，被调用的函数一般是不加锁的。当然你要仔细考量，保证互斥。
