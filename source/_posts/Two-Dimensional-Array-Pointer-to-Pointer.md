---
title: Two-Dimensional Array & Pointer to Pointer
tags:
  - C/C++
  - 指针
categories:
  - 编程语言
  - C/C++

date: 2018-10-29 10:54:50
---

&emsp;First of all, let's consider the difference of three functions as follow:    

1.  void write(char a)

2.  void write(char *a)

3.  void write(char **a)    
<!--more-->
&emsp;I am sure you have seen 1 and 2 functions, they respectively are __passing-by-value__ and __passing-by-address__. The input of 1 can only be a __char__, the input of 2 can be a __pointer to char__ or a __char array__. Both of them are easy to understand. But have you seen function 3? It's confusing some times, because the input of 3 can be a __pointer to pointer that point to char__ or a __pointer to char array__ or a __two dimensional char array__. It is different when you code a function like 3, the difference as follows:

```c++
//bm->read_block(unint32_t block_num,char *buf)
//copy memory from char blocks[block_num][] by memcpy()


//case 1
void inode_manager::read_file(uint32_t inum, char **buf_out, int *size)
{//when input of char **buf_out is two-dimensional char array which is declaration as char buf_out[BLOCK_NUM][BLOCK_SIZE]
        struct inode *file_inode = get_inode(inum);
        *size =file_inode->size;
        int len;
        int i;
        len = getFileBlockNum(*size);
        for(i = 0;i < len;i++)
        {
            bm->read_block(file_inode->blocks[i],buf_out[i]);//or
            //bm->read_block(file_inode->blocks[i],(char *)buf_out + (i * BLOCK_SIZE));
        }
    	
}

//case 2
void inode_manager::read_file(uint32_t inum, char **buf_out, int *size)
{//when input of char **buf_out is pointer to char array which is declaration as char *buf_out; and call this function like read_file(id,&buf_out,size)
    struct inode *file_inode = get_inode(inum);
    *size =file_inode->size;
    int len;
    int i;
    len = getFileBlockNum(*size);
    *buf_out = (char *)malloc(MAXFILE * BLOCK_SIZE);
    for(i = 0;i < len;i++)
    {
      bm->read_block(file_i->blocks[i],((*buf_out) + i*BLOCK_SIZE));
    }
  }


//case 3
//when the input of char **buf_out is pointer to pointer that pointe to char,which is declaration as char **buf_out. What you can do to the input is rare.
```

&emsp;As to case 2,do you think the following code is correct?

```c++
void inode_manager::read_file(uint32_t inum, char **buf_out, int *size)
{
        struct inode *file_inode = get_inode(inum);
        *size =file_inode->size;
        int len;
        int i;
        len = getFileBlockNum(*size);
        for(i = 0;i < len;i++)
        {
            buf_out[i] = (char *)malloc(sizeof(char) * BLOCK_SIZE);
            bm->read_block(file_inode->blocks[i],buf_out[i]);//the following line is alse wrong
            //bm->read_block(file_inode->blocks[i],(char *)buf_out + (i * BLOCK_SIZE));
        }
    	
}
//bm->read_block(unint32_t block_num,char *buf)//copy memory from char blocks[block_num][] by memcpy()
```

&emsp;The answer is incorrect. Do you know the reason?
