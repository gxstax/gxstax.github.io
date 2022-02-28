---
title: linux编程-文件读取操作
date: 2019-05-28 10:21:56
categories:
    - linuxProgram
tags: [linuxProgram]
---

### 这里写一个最简单的文件读取的操作流程：

1. 安装环境

   我的系统是centos7，默认是不带有linux函数系统手册的，所以有需要的要先安装一下。具体的命令如下：

   ```bash
   $ yum -y install man-pages
   ```

   出现如下提示，说明安装成功；

   ```bash
   $ Running transaction 正在安装 : man-pages-3.53-5.el7.noarch
   $ 1/1 验证中 : man-pages-3.53-5.el7.noarch
   $ 1/1 已安装: man-pages.noarch 0:3.53-5.el7
   $ 完毕！
   ```

2. 开始着手写我们的程序

   + 文件读取操作

     我们这里随便建一个c程序，就叫open_test1.c,首先写一个main函数

     ```c
     int main(void)
     {
     	return 0;
     }
     ```

     下面我们需要查询手册，来看linux系统是怎么打开一个文件的；这里我们要使用man 2 open 命令查看：

     ```bash
     $ man 2 open
     ```

     > OPEN(2)                     BSD System Calls Manual                    OPEN(2)
     >
     > NAME
     >      open, openat -- open or create a file for reading or writing
     >
     > SYNOPSIS
     >      #include <fcntl.h>
     >
     >      int
     >      open(const char *path, int oflag, ...);
     >     
     >      int
     >      openat(int fd, const char *path, int oflag, ...);
     >
     > DESCRIPTION
     >      The file name specified by path is opened for reading and/or writing, as specified by the
     >      argument oflag; the file descriptor is returned to the calling process.
     >
     > ```bash
     >  The oflag argument may indicate that the file is to be created if it does not exist (by speci-
     >  fying the O_CREAT flag).  In this case, open() and openat() require an additional argument
     >  mode_t mode; the file is created with mode mode as described in chmod(2) and modified by the
     >  process' umask value (see umask(2)).
     >  省略部分内容..........
     > ```

     具体的描述如如上面所示，所以我们根据他给我们的提示，写一个打开文件的操作，我们的程序中增加如下代码：

     ```c
     #include <stdio.h>
     #include <sys/types.h>
     #include <sys/stat.h>
     #include <fcntl.h>
     
     int main(void)
     {
             int fd = 0;
             fd = open("./file.txt",O_RDWR);
             if(-1 == fd)
             {
                     printf("open fail\n");
                     return 0;
             }
             else
             {
                     printf("open ok\n");
             }
             
              // 关闭文件
             close(fd);
     }
     ```

     如果文件打开正常，会打印出，open ok；

     

   + 写文件操作

     同样的我们去查写操作手册，命令为：

     ```bash
     $ man 2 write
     ```

     > WRITE(2)                    BSD System Calls Manual                   WRITE(2)
     >
     > NAME
     >      pwrite, write, writev -- write output
     >
     > LIBRARY
     >      Standard C Library (libc, -lc)
     >
     > SYNOPSIS
     >      #include <unistd.h>
     >
     >      ssize_t
     >      pwrite(int fildes, const void *buf, size_t nbyte, off_t offset);
     >     
     >      ssize_t
     >      write(int fildes, const void *buf, size_t nbyte);
     >     
     >      #include <sys/uio.h>
     >     
     >      ssize_t
     >      writev(int fildes, const struct iovec *iov, int iovcnt);
     >
     > DESCRIPTION
     >
     > 省略部分内容.....................

     所以我们的程序中增加代码如下：

     ```c
     #include <stdio.h>
     #include <sys/types.h>
     #include <sys/stat.h>
     #include <fcntl.h>
     #include <unistd.h>
       
     int main(void)
     {
             int fd = 0;
             // 打开file.txt文件
             fd = open("./file.txt",O_RDWR);
     
             if(-1 == fd)
             {
                     printf("open fail\n");
                     return 0;
             }
             else
             {
                     printf("open ok\n");
             }
             
             // 往file.txt文件里面写入hello world
             char buf[] = "hello world";
             write(fd,(void *)buf,11);
     
             //ssize_t write(int fd, const void *buf, size_t count);
     
     	    // 关闭文件
             close(fd);
     }
     
     
     ```

     

   + 读取写入文件的到缓冲区，然后输出

     读取file.txt文件里的内容到buf2中，然后输出，同样我们查看读文件的函数命令: 

     ```bash
     $ man 2 read
     ```

     > READ(2)                     BSD System Calls Manual                    READ(2)
     >
     > NAME
     >      pread, read, readv -- read input
     >
     > LIBRARY
     >      Standard C Library (libc, -lc)
     >
     > SYNOPSIS
     >      #include <sys/types.h>
     >      #include <sys/uio.h>
     >      #include <unistd.h>
     >
     > ```bash
     >  ssize_t
     >  pread(int d, void *buf, size_t nbyte, off_t offset);
     > 
     >  ssize_t
     >  read(int fildes, void *buf, size_t nbyte);
     > 
     >  ssize_t
     >  readv(int d, const struct iovec *iov, int iovcnt);
     >  省略剩余内容.....................
     > ```

     然后我们的程序增加如下：

     ```c
     #include <sys/types.h>
     #include <sys/stat.h>
     #include <fcntl.h>
     #include <unistd.h>
     #include <stdio.h>
     
     int main(void)
     {
             int fd = 0;
             // 打开file.txt文件
             fd = open("./file.txt",O_RDWR);
     
             if(-1 == fd)
             {
                     printf("open fail\n");
                     return 0;
             }
             else
             {
                     printf("open ok\n");
             }
             // 往file.txt文件里面写入hello world
             char buf[] = "hello world";
             write(fd,(void *)buf,11);
     
             //ssize_t write(int fd, const void *buf, size_t count);
     
             // 把文件指向调到文件头部位置
             lseek(fd, SEEK_SET, 0);
             //off_t lseek(int fd, off_t offset, int whence);
     
             // 读取file.txt的内容到buf2中，然后输出
             char buf2[30] = {0};
             read(fd, buf2, sizeof(buf2));
     
             printf("buf2 = %s\n", buf2);
     
             // 关闭文件
             close(fd);
             
             return 0;
     }
     
     ```

     到此为止，我们的程序基本就算是写完了！

     

3. 运行

   + 那么让我们编译运行一下：

     ```bash
     $ gcc open_test1.c
     ```

     如果没有报错，则说明我们的程序没有问题：编译后会出现一个a.out文件如下：

     ```bash
     $ ll
     总用量 20
     -rwxr-xr-x. 1 root root 8704 5月 28 10:19 a.out
     -rw-r–r--. 1 root root 11 5月 28 09:52 file.txt
     -rw-r–r--. 1 root root 757 5月 28 10:12 open_test1.c
     ```

     然后让我们运行一下看一下结果：

     ```bash
     $ ./a.out
     open ok
     buf2 = hello world
     ```

     成功输出！！！

     

到这里一个简单的文件打开->写入->输出的简单操作就完成了。

