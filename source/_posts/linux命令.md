---
title: linux命令
date: 2019-04-13 17:07:28
categories:
    - linux
tags: [linux]
---

### Linux基本命令

1. 目录示意
   * /:根目录
     整个文件系统，有一个顶层目录，称为根。 bin:存放一些可执行的程序、命令。
   * boot: 系统启动所需的一些文件。
   * dev:系统中的设备(硬件在linux中通过“文件”来标识) etc:存放系统、软件的配置文件
   * home:普通用户目录的主目录，以用户名命名。
   * home/fred lib:系统库目录(32位)
   * lib64: 系统库目录(64位)
   * media:媒体
   * mnt:挂载外部存储设备的文件目录
   * opt
   * proc
   * root:root用户的主目录
   * run
   * sbin:系统的可执行命令
   * srv
   * sys
   * test
   * tmp:系统临时目录 usr:共享资源目录(多个用户可以共享该目录中的程序)
   * var

2. 命令

   * ls命令

     ```bash
     ls /: 查看根目录
     ls -l: 显示详细信息
     ls -lh: 显示跟符合人类查看方式 
     ls -a: 显示隐藏文件
     ```

     

   * 目录切换

     ```bash
     pwd: 查看当前所在目录
     cd: 切换目录
     cd ...: 退回到上一级目录cat
     ```

     

   * 创建文件夹

     ```bash
     mkdir aaa: 相对路径写法
     mkdir /bbb: 绝对路径写法
     mkdir -p aaa/bbb/ccc: 级联创建目录
     ```

     

   * 删除目录/文件

     ```bash
     rm: 删除目录
     rm -r: 递归删除
     rm -rf: 递归删除，不提示
     ```

     

   * 查看信息

     ```bash
     touch: 创建空文件
     cat: 查看文件内容
     ```

     

   * 编辑

     ```bash
     vi : 编辑文件
     -i: 编辑模式 
     -o: 编辑模式(直接到下一行)
     -w: 保存
     -q: 退出
     esc: 退出编辑 
     ```

     

   * Vim快捷键 (非编辑模式下)

     ```bash
     a: 在光标后一位开始插入
     A: 在该行的最后插入
     I: 在该行的最前插入
     yy: 复制整行 
     3yy: 复制三行
     p: 粘贴
     gg: 直接跳到文件首行
     G: 直接跳到文件的末行
     dd: 删除一行
     3dd: 删除三行
     /: 搜索内容,n匹配下一个 u:undo(撤销) ctrl+r:redo(执行之前撤销的) :set nu: 设置行号
     :set nonu: 设置不显示行号
     :q!: 强制不保存退出
     fg 程序编号: 切换后台挂起程序
     jobs: 查看后台挂起的程序
     ctrl+z: 将程序挂起
     ```

     

   * 拷贝

     ```bash
     cp a.txt b.txt: 将a.txt 拷贝为b.txt
     ```

     

   * 移动/改名

     ```bash
     mv a.txt aa.txt: 将a.txt 改为 aa.txt
     mv a.txt aa/aa.txt: 将a.txt 移动到 aa/aa.txt
     ```



 

 3. 权限

    * 添加用户

      ```bash
      useradd fred passwd 1234: 添加 fred 用户并设置密码为 1234
      ```

      

    * linux文件权限的描述格式

      ```bash
      d rwx rwx rwx
      d: 标识节点类型(d:文件夹 -:文件 |:链接)
      r: 可读
      w: 可写
      x: 可执行
      第一组rwx: 表示这个文件的拥有者对它的权限
      第二组rwx: 表示这个文件的所属组用户对它的权限
      第三组rwx: 表示这个文件的其他用户(除以上两种)对它的权限
      使用二进制表示权限:例如-rw-rw-r–二进制表示为110,110,100，十进制表示为664
      
      补充:
      r: 对文件来说，是可读取内容;对文件夹来说，是可以ls
      w: 对文件来说，是可修改文件的内容;对文件夹来说，是可以在其中创建或者删除子节点
      x: 对文件来说，是能否运行这个文件;对文件夹来说，是能否cd进入这个目录
      ```

      

    * 用户管理

      * 增加用户

        ```bash
        useradd ant: 增加 ant 用户
        passwd ant: 给用户ant设置密码
        userdel -r 用户名: 删除用户(加一个-r表示把用户及用户的主目录都删除)
        exit: 退出会话
        ```

        

      * 增加用户组

        ```bash
        groupadd 组名: 增加组
        usermod -g 组名 用户名: 将用户添加到组中
        usermod -G 组名1,组名2 用户名: 将用户添加到多个组中
        gpasswd -d 用户名 组名: 将用户从组中删除
        	—例如:gpasswd -d jack root | gpasswd -d jack sys
        ```

        

      * 查看所属组

        ```bash
        groups: 查看当前用户所属组 
        groups jack: 查看指定用户所属组
        ```

        

      * su 和 sudo

        ```bash
        su: 身份切换
        su username 输入密码(root切换不需要输入密码)
        sudo: 让普通用户具备root的权限(需要配置 /etc/sudoers)
        
        ```

        > 了解完su和sudo，是不是发现sudo有太多的优点了。
        >
        > su方式切换是须要输入目标用户的password。而sudo仅仅须要 输入自己的password，所以sudo能够保护目标用户的password不外流的。
        >
        > 当帮root管理系统的时候，su是直接将 root全部权利交给用户。而sudo能够更好分工，仅仅要配置好/etc/sudoers，这样sudo能够保护系统更安全，并且分 工明白，有条不紊。

        


