---
title: Linux用户权限与文件系统
date: 2020-07-09 20:42:44
tags: 
- Linux
categories: 
- Linux
---

<center>
引言：

Linux用户权限与文件系统
</center>

<!-- more -->

# Linux文件系统


## 用户与用户组

> linux中每一个文件都有三个权限身份`User、Group、Others` 

`User`就是指用户，每一名用户对文件的操作权限都不同

`Group`就是用户组，对一个文件，用户组内对其操作权限相同

`Others`，对于一个用户来说，其他用户对其就是`Others`，其他人

`root`对于linux来说是神，拥有最高权限

> 在Linux中，所有用户的账号与root用户信息都是在`/etc/passwd`，用户组信息存放在`/etc/group`，个人的密码存放在`/etc/shadow`



-----

## Linux文件属性

使用`ls -al`来查询文件的类别

```shell
[root@localhost usr] ls -al
drwxr-xr-x.  17 root root   221 Jul  4 19:33 .
dr-xr-xr-x.  17 root root   224 Jul  5 18:17 ..
dr-xr-xr-x.   2 root root 49152 Jul  4 19:35 bin
drwxr-xr-x.   3 root root    79 Jul  4 19:16 cmake
drwxr-xr-x.   2 root root     6 Apr 10  2018 etc
lrwxrwxrwx.   1 root root    10 Jul  4 01:47 tmp -> ../var/tmp
```

总共分为七个部分：

- 第一个部分`dr-xr-xr-x`：
  - 第一个字符有多种不同的文件类型
    - `[d]`：目录
    - `[-]`：文件
    - `[l]`：链接，并且在文件名会显示出链接到哪里
    - `[b]`：可供存储的接口设备
    - `[c]`：串行端口设备（例如鼠标键盘）
  - 后九个字符，每三个一组，分别代表`User`、`Group`、`Others`对该文件的操作权限
    - `[r]`：read，可读
    - `[w]`：write
    - `[x]`：execute
- 第二个部分`17`，这个数字代表连接数，有多少文件名连接到此节点
- 第三个部分代表文件的所有者`User`账号
- 第四个部分代表文件的所属用户组`Group`
- 第五个部分代表文件的大小，单位是`B`字节
- 第六个部分代表文件的创建时间或者文件的最近更新时间
- 第七个部分代表文件名，文件名前有`.`，代表这个文件为隐藏文件

## 改变文件的操作权限

该笔那文件的操作权限，有以下三个命令

```shell
chgrp # 改变用户组
chown # 改变用户
chmod # 改变文件的权限
```

### 改变用户组的操作权限

`chgrp -R [用户组名] [文件名]`

`-R`参数代表的意思递归的持续更改，连同文件下的子目录

```shel
[root@localhost usr]# ls -l text.txt 
-rw-r--r--. 1 root root 0 Jul  8 19:08 text.txt
[root@localhost usr]# chgrp usr text.txt 
[root@localhost usr]# ls -l text.txt 
-rw-r--r--. 1 root usr 0 Jul  8 19:08 text.txt
```

此处的用户组名是`usr`，用户组名必须是`/etc/group`目录下的文件，如果在`/etc/group`目录下不存在用户组会报错

```
[root@localhost usr]# chgrp xxx text.txt 
chgrp: invalid group: ‘xxx’
```

### 改变用户的操作权限

`chown -R [账号名] [文件名]`

```shell
[root@localhost usr]# ls -l text.txt 
-rw-r--r--. 1 root usr 0 Jul  8 19:08 text.txt
[root@localhost usr]# chown usr text.txt
[root@localhost usr]# ls -l text.txt 
-rw-r--r--. 1 usr usr 0 Jul  8 19:08 text.txt
```

同样，此处的`usr`必须是`/etc/passwd`文件下有的，否则也会报错

### 改变文件的操作权限（重点）

Linux中共有三个身份，每个身份对应三个权限，分别是`rwx`，现在分别给各个权限一个数字

```
r → 4
w → 2
x → 1
- → 0
```

那么假设有这样一个操作权限组`-rw-r--r--`，那么他的操作权限数字就是622（4+2+0,2+0+0,2+0+0）

那么最高权限就是777了

```shell
[root@localhost usr]# ls -l text.txt 
-rw-r--r--. 1 usr usr 0 Jul  8 19:08 text.txt
[root@localhost usr]# chmod 777 text.txt 
[root@localhost usr]# ls -l text.txt 
-rwxrwxrwx. 1 usr usr 0 Jul  8 19:08 text.txt
```

------

除了给数字，我们还可以通过字符给文件设置可读可写

```shell
chmod  u/g/o/a +/-/= r/w/x [文件名]
# 给user/group/other 增添/减去/设置 读/写/执行
```

例如：

```shell
[root@localhost usr]# ls -l text.txt 
-rwxrwxrwx. 1 usr usr 0 Jul  8 19:08 text.txt
[root@localhost usr]# chmod u=rw,go-wx text.txt 
[root@localhost usr]# ls -l text.txt 
-rw-r--r--. 1 usr usr 0 Jul  8 19:08 text.txt
```

## 对于文件和目录，rwx分别对应着什么

### 对于文件

```shell
r	代表用户可以读文件的内容
w	代表用户可以更改增添文件的内容
x	代表用户可以执行这个文件
```

注意：

1. 对文件给予`w`权限，不代表用户可以删除这个文件
2. 在Linux系统中判断文件是否可以执行，不同于windows中的`.exe`文件

### 对于目录

```
r	代表用户使用ls查询这个目录下的内容
w	代表用户可以更改这个目录下的文件结构，可以删除，可以创建
x	代表用户可以cd到这个目录下
```

注意：

1. `w`的权限不能轻易给

## Linux中的文件种类与扩展名

### 文件种类：

1. 普通文件（regular file），属性用`[-]`表示
2. 纯文本文件（ASCII），可以使用`cat`直接读取数据
3. 二进制文件（binary），可执行程序就是二进制文件
4. 数据格式文件（data），程序在运行过程中需要读取特定格式的文件，例如登录数据`/var/log/wtmp`只能使用`last`读取，使用`cat`读取会乱码
5. 目录（directory），属性使用`[d]`表示
6. 连接文件（link），类似windows中的快捷方式
7. 设备与设备文件（device），与外设或存储有关的文件，集中在`/dev`目录下，分为两种
   1. 块（block）设备文件：提供一些存储数据的接口设备，属性用`[b]`表示
   2. 字符（character）设备文件：串行端口的接口设备，属性为`[c]`
8. 套接字（Socket），数据接口文件，网络上的数据连接，属性为`[S]`
9. 管道（FIFO，pipe），解决多个程序同时访问到一个文件时所造成的错误，属性为`[P]`

### 文件扩展名

在linux中文件扩展名没有作用，这也是和windows重要的区别之一，在linux中的操作权限由`rwx`来决定，与`.exe .bat`等等是没有任何关系的，在Linux中后缀名的唯一作用就是告诉你一些关于这个文件的信息

1. `.sh`：脚本或批处理文件（scripts）
2. `.tar .xz .zi .tar.gz`等：压缩文件
3. `.html .php`网页先关文件

### 文件名长度的限制

linux使用默认的`Ext2/Ext3`文件系统：

- 文件或目录最大容许文件名最大容许255个字符
- 包含完整路径及目录(/)的完整文件名为4096个字符
- 避免使用特殊字符`* ? < > ; % ^ & * ( ) {  }`

## 文件目录配置

| 目录     | 应放置的文件内容                                             |
| -------- | ------------------------------------------------------------ |
| `/bin`   | 放置执行文件。在bin下的命令可以被root或是一般用户使用        |
| `/boot`  | 放置开机会用到的文件。包括内核的引导文件、引导程序（grub）   |
| `/dev`   | 放置设备和接口设备的文件。                                   |
| `/etc`   | 放置系统的配置文件。重要的文件有：<br />1. `/etc/init.d`：放置所有服务的默认脚本<br />2. `/etc/X11`：放置与 X Window 有关的各种文件配置<br />3. `/etc/xinetd.d`：super daemon（总管进程）管理的各项服务的配置文件目录 |
| `/var`   | 单独来谈                                                     |
| `/home`  | 一般用户的用户主目录。有两种表示方法：1. `~` 2. `~[用户名]`  |
| `/usr`   | 重要目录，专门来谈                                           |
| `/lib`   | 放置开机、运行`/bin`或是`/sbin`目录下命令的函数库。尤其是`/lib/modules`下，放置着linux内核相关的模块 |
| `/media` | 放置着可删除设备，例如光盘、软盘                             |
| `/opt`   | 放置第三方程序。不过还是习惯放置在`/usr/local`目录下         |
| `/root`  | `root`的主文件夹，如果在`root`下`cd ~`，会进入到这个文件夹下 |
| `/sbin`  | 放置开机所需要的重要命令。服务器软件程序产生的命令放置在`/usr/sbin`，系统自行安装的软件的命令放置在`/usr/local/sbin`下 |
| `/srv`   | 一些网络服务启动后，这些服务所需要取用的数据目录，例如WWW，FTP服务 |
| `/tmp`   | 一般用户或者是正在执行的程序暂时存放文件的地方               |
| `/proc`  | 虚拟文件系统，放置的数据都在内存当中，不占硬盘空间           |
| `/sys`   | 类似于`/proc`                                                |

其中，开机主要和根目录有关，其他分区是在开机完成后才会持续进行挂载的行为，所以有五个目录不能分区到其他硬盘

1. `/etc`配置文件
2. `/bin`可执行文件
3. `/dev`所需要的设备文件
4. `/lib`执行文件所需要的函数库与内核所需要的模块
5. `/sbin`重要的系统执行文件

----

文件目录配置标准（FHS）

四种交互形态：

1. 可分享与不可分享
   1. 可分享：可以分享给其他系统使用的目录
   2. 不可分享：与本机有关，不可分享的
2. 可变动的与不可变动
   1. 可变动的：经常改变的数据
   2. 不变的：有些数据不会跟着`distribution`的变动而变动，例如函数库等

![img](http://img.yesmylord.cn/adsjflkasdjfkasjdlfasdfa.png/999)

---

### `/usr`目录

1. 由FHS，可以知道`/usr`是可分享不可变动的。
2. `usr`不是`user`的缩写，是Unix Software Resource（Unix操作系统软件资源）的缩写

| 目录           | 应放置文件内容                                               |
| -------------- | ------------------------------------------------------------ |
| `/usr/X11R6`   | 为X Window系统重要数据所放置的目录                           |
| `/usr/bin`     | 绝大多数用户的命令都可以放在这里，与`/bin`的区别是，没有与开关机相关的命令 |
| `/usr/include` | C/C++语言的头文件                                            |
| `/usr/lib`     | 包含各应用的函数库、目标文件、不被一般用户使用的命令         |
| `/usr/local`   | 用户在本机自行安装下载的软件                                 |
| `/usr/sbin`    | 非系统正常运行所需要的命令（网络服务器软件的服务命令）       |
| `/usr/share`   | 可以分享的与平台无关的文件                                   |
| `/usr/src`     | 放置源码，linux内核的源码就在`/usr/src`中                    |

### `/var`目录

| 目录         | 应放置文件内容                                           |
| ------------ | -------------------------------------------------------- |
| `/var/cache` | 放置应用程序中产生的一些暂存文件                         |
| `/var/lib`   | 放置应用程序运行中需要使用到的数据放置的目录             |
| `/var/lock`  | 某些文件一次只能被一个程序所使用，因此就得给这种文件上锁 |
| `/var/log`   | 存放登录文件                                             |
| `/var/mail`  | 存放个人邮件                                             |
| `/var/run`   | 某些程序或是服务启动后，PID会存放在这个目录下            |
| `/var/spool` | 队列数据（排队等待其他应用使用的数据）                   |

---

其他

`.`与`..`

- `.`代表当前目录，也可以使用`./`表示
- `..`代表上一层目录，也可以使用`../`来表示