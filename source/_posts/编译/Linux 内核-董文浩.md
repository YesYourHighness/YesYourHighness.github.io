---
title: Linux内核编译
date: 2020-07-07 10:18:49
tags:
- Linux
categories: 
- Linux
---



<center>
引言： 

编译linux内核

</center>

<!-- more -->


# Linux内核编译



## 前置要求

[官网下载](https://www.kernel.org/)压缩文件

新建一个文件夹存放这个压缩文件，这里新建的文件夹为`/usr/kernel`，将文件解压到此`tar -xf xxxx.gx.xz`

## 开始配置安装

1. 进入到`linux`内核路径下

   如果是不是第一次执行，则要清理一下之前残留的配置文件

   ```shell
   make mrproper
   # 如果不是第一次配置，运行此命令后会显示删除xxx.conf文件等等
   make clean
   ```

2. 拷贝系统原有的操作模板

   ```shell
   cp /boot/config-xxx-xx [我们解压后的linux路径/.config]
   ```

   定义编译内核时功能的特性

3. 安装需要的组包

   ```shell
   yum groupinstall "development tools" 
   ```

4. 配置内核选项，这里有多种配置

   ```shell
   make defconfig
   # 默认配置（据说是linus的配置）
   
   make allnoconfig
   # 只安装必须安装的选项（适用嵌入式系统）
   
   make menuconfig
   # 图形化界面安装方式（需要ncurses-devel，记得先yum install）
   ```

5. 这里选择了`make menuconfig`方式（如果报错`display`不够什么的错误，就调大一点你的窗口，就ok了）

   1. 进入General setup
   2. 我们改一下`Local version -append to kernel release`
   3. 加上自己的名字吧`-1.0-wenhaoLinux`，点击OK
   4. 一步一步`exit`，选择`YES`保存退出即可

   使用`grep -i ntfs .config`查询一下我们刚刚所做的配置

6. 开始编译，`make -j 4`，需要很长的一段时间，如果报错说缺少什么，直接`yum即可`
7. 开始安装模块`make modules_install`，安装后`ls /lib/modules`查看咱们自己编译的内核
8. `make install`安装内核相关的文件，安装后使用`ls /boot`查看，会有三个我们自己产生的文件`initramfs、System.map、vmlinuz`
9. 查询`grub`菜单，`cat /boot/grub2/grub.cfg`查看，会有对应我们新装的内核的菜单
10. `reboot`，重新启动，在启动页可以看到我们自己安装内核的`Linux`

![img](http://img.yesmylord.cn/Y2M0G9UU.png)

11. 进入后，使用`uname -a`查看，发现确实是我们的版本

    ![img](http://img.yesmylord.cn/dasjdaklsd.png)

    