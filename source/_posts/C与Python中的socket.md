---
title: C与Python中的socket
tags:
  - C
  - Python
  - socket
  - 网络编程
categories:
  - 学习
  - 计算机及软件
  - 网络编程
date: 2019-11-15 10:53:07
---


# C与Python中的socket

本文主要是想实现一下C与Python的socket通信，顺便说一下两者各自的socket编程。所以全篇结构如下：

* C中的socket
* Python中的socket
* C与Python的socket通信

<!--more-->

## C 中的socket

这部分主要参考《UNIX环境高级编程（第3版）》

```c
//server.c
#include <stdio.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define BUFF_SIZE 100
#define HOST "127.0.0.1"
#define PORT 65432
#define MSG "Hello FROM C Server"

int main(int argc, char* argv[])
{
    int server_fd;
    int client_fd;

    char buf[BUFF_SIZE];

    struct sockaddr_in local_addr;
    struct sockaddr_in remote_addr;

    memset(&local_addr, 0, sizeof(local_addr)); //清零local_addr
    local_addr.sin_family = AF_INET;
    local_addr.sin_addr.s_addr = inet_addr(HOST);
    local_addr.sin_port = htons(PORT);

    if((server_fd = socket(PF_INET, SOCK_STREAM, 0)) < 0 ) // 新建socket
    {
        perror("socket:");
        return -1;
    }

    if((bind(server_fd, (struct sockaddr *)&local_addr, sizeof(struct sockaddr))) < 0) // 绑定socket
    {
        perror("bind:");
        return -1;
    }
    listen(server_fd, 10); // 监听socket
    while(1)
    {
        int sin_size = sizeof(struct sockaddr_in);
        if( (client_fd = accept(server_fd, (struct sockaddr *)&remote_addr, &sin_size )) <0 ) // 接受client的链接请求
        {
            perror("accept:");
            return -1;
        }
        pid_t handle_pid;
        handle_pid = fork();
        if(handle_pid == 0) // 子进程处理请求
        {
            if(close(server_fd) < 0)
            {
                perror("close:");
                return -1;
            }
            int len = send(client_fd, MSG, sizeof(MSG),0);
            printf("len = %d\n",len);
            if(len != sizeof(MSG))
            {
                perror("send:");
                return -1;
            }
        }
        if(close(client_fd) < 0) // 父进程关闭client的socket，继续监听。
        {
            perror("close:");
            return -1;
        }
        exit(0);
    }
    if(close(client_fd) < 0)
    {
        perror("close:");
        return -1;
    }

    return 1;
}
```

```c
//client.c
#include <stdio.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define BUFF_SIZE 100
#define HOST "127.0.0.1"
#define PORT 65432

int main(int argc, char* argv[])
{
    int server_fd;
    struct sockaddr_in server_addr;
    char buf[BUFF_SIZE];
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = inet_addr(HOST);
    server_addr.sin_port = htons(PORT);
    if((server_fd = socket(PF_INET, SOCK_STREAM, 0)) < 0 ) // 新建socket
    {
        perror("socket:");
        return -1;
    }

    if((connect(server_fd, (struct sockaddr*)&server_addr, sizeof(struct sockaddr)) < 0) ) // 连接server
    {
        perror("connect:");
        return -1;
    }
    int len = recv(server_fd, buf, BUFF_SIZE, 0); //连接成功后接收信息
    while(len > 0)
    {
        for(int i = 0; i < len; i++)
        {
            putchar(buf[i]);
        }
        putchar('\n');
        len = recv(server_fd, buf, BUFF_SIZE, 0);
    }
    close(server_fd);
    return 1;
}
```



## Python 中的socket

```python
#server.py
import socket

HOST = '127.0.0.1'
PORT = 65432

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
  s.bind((HOST, PORT))
  s.listen(5)
  while True:
    conn, addr = s.accept()
    with conn:
      conn.send(b"Hello From Python Server!")
      conn.close()
```

```python
#client.py
import socket
HOST = '127.0.0.1'
PORT = 65432

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
  s.connect((HOST, PORT))
  msg_part = s.recv(10)
  msg = msg_part
  while len(msg_parg) == 10:
    msg += msg_parg
    msg_parg = s.recv(10)
  msg += msg_part
  print(msg)
```



## C与Python的socket通信

C和Python通过套接字是可以直接通信的。需要注意的是消息的格式。socket传输的内容是byte流，C中可以使用强制类型转换，Python中则需要`pack()`与`unpack()`。

比如：

在C写的server端发送：

```c
int msg = htonl(999);
send(client_fd, (const void *)&msg, sizeof(msg),0);
```

Python端接收：

```python
from struct import *
(msg,) = unpack('>1I',socket.socket.recv(4))
print(msg)
```

`unpack()`的第一个参数表示格式，`>1I`中的`>`表示大端（相应的`<`就表示小端），`1I`表示一个整数。

`pack()`与`unpack()`的详细操作见*博主 **三月沙** 的《[Python 中的 pack 和 unpack](https://sanyuesha.com/2018/03/10/why-pack-unpack/)》*。



---

反过来也是一样：

在Python写的server中发送

```python
conn.send(pack('>I',999))
```

C写的client中接收：

```c
int msg;
int len = recv(server_fd, (int *)&msg, sizeof(int),0);
printf("%d\n",ntohl(*msg));
```

