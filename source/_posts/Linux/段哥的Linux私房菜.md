---
title: 段哥的Linux私房菜
date: 2020-07-12 10:06:44
tags: 
- Linux
categories: 
- Linux
---
<center>
引言：

段哥的Linux私房菜：分区与挂载、网络操作、日志查看

</center>

<!-- more -->

# 段哥的Linux私房菜



## 分区与挂载操作

使用VirtualBox创建一个磁盘，并且添加到虚拟机上

### 创建分区

`fdisk`（磁盘管理：创建分区、删除分区、查看分区）

```shell
# 管理磁盘
fdisk /dev/sdb
# m 可以获取帮助
# n 创建一个新的分区  -> p创建主分区 e创建扩展分区
# p 打印当前创建的分区
# w 保存并退出
# q 不保存退出
```

创建完成后，生成了sdb1，我们就分好了一个区

### 格式化

格式化就是给当前分区头部添加一段数据，这段数据标明这个分区所使用的文件格式等信息。

```shell
mkfs.ext4
# 有多种文件系统可以选择 例如 ext4 xfs cramfs 等等
# 这里选择了经典的ext4
```

### 挂载

在格式化完成后，就可以挂载目录了，接下来我们给`/dev/sdb1`分区挂载一个目录

首先建立一个目录

```
mkdir /data
```

使用`mount`命令挂载

```shell
mount /dev/sdb1 /data
```

然后使用`df -h`查看挂载情况，我们就挂载好了

### 卸载

在卸载之前，我们可以向这个分区建立一个文件，例如

```shell
cd /data
# 进入到挂载点
touch 1.txt
# 创建一个新文件
```

卸载了它

```shell
umount /dev/sdb1
# 注意，如果报错buzy，意思是你不能在挂载点进行卸载操作，你可以换一个地方
```

再使用`df -h`查看，我们就卸载了这个挂载点

如果我们再装载上它，会发现，数据都还是在的，挂载只是将从根延伸出来的一个目录安装到一个分区上，（由于磁盘大小的限制，我们不可能将所有的根下的目录都挂载在同一个地方），从根延伸出来的这个目录，和根目录存储的分区不同（例如这个例子，我们的`/`挂载在`/dev/sdb1`，而我们将`/data`挂载在`/dev/sdb2`上）

## 网络操作

局域网内，通过mac地址就可以找到

广域网需要通过网关，不配置网关不能访问外网

DNS服务器：域名转换为IP，当我们输入一个域名，DNS会自动将这个域名换为IP地址

GFW(Great FireWall)：墙，通过污染DNS，影响网站返回的真实IP地址，这就是墙



实体机和虚拟机连接：能互相访问即可

windows方面：

```shell
ipconfig
# 查看ip
```

linux方面：

```shell
ip addr
# 查询ip地址
```

![image-20200711203819337](http://img.yesmylord.cn//image-20200711203819337.png/999)

可以看到有两张网卡：

1. `lo`（Loopback Address）：回环地址，不属于任何一个有类别地址类。代表设备的本地虚拟接口，所以默认被看作是永远不会宕掉的接口

2. `enp`：以太网

   ```shell
   # 默认关闭,查看配置文件
   cat /etc/sysconfig/network-scripts/ifcfg-enp0s
   ```

   ![image-20200711204745385](http://img.yesmylord.cn//image-20200711204745385.png/999)

   注意到`ONBOOT=no`默认开机不会启动，而且`BOOTPROTO=dhcp`，`dhcp`是一个自动获取ip地址的协议，我们需要让他启动

   >DHCP（动态主机配置协议）是一个局域网的网络协议。指的是由服务器控制一段IP地址范围，客户机登录服务器时就可以自动获得服务器分配的IP地址和子网掩码。默认情况下，DHCP作为Windows Server的一个服务组件不会被系统自动安装，还需要管理员手动安装并进行必要的配置。（百度百科）

   ```shell
   dhclient
   # 启动dhcp
   ```

连接方式：

1. 桥接：直接与电脑使用同一台路由器（与电脑的网卡获取相同的路由器IP来源，共用同一个IP，同一个带宽）
2. NAT：与主机共用同一个IP，但是在内部模拟了一个虚拟的IP，主机和虚拟机不是一个IP
3. Host-only：与宿主机形成一个内部的小网络，主机和虚拟机可以互相访问，但是不能访问外网

在启动dhc服务之后，虚拟机根据不同的连接方式，发送给网卡不同的请求。

（在本机实验时，使用NAT方式，发现虚拟机可以ping通主机，但是主机ping不通虚拟机。所以改用了Host-only方式，但是发现在Virtual Box环境下，报错

```shell
Failed to open/create the internal network 'HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter' (VERR_INTNET_FLT_IF_NOT_FOUND).
Failed to attach the network LUN (VERR_INTNET_FLT_IF_NOT_FOUND).
```

后来在网络与共享中心中更改适配器设置，将Host-only那张网卡禁用，再重新启动，即可解决）

在主机ping通虚拟机后，就可以使用（Xshell、Termius）SSH服务远程操纵虚拟机了



## 操作技巧

### 重定向

```shell
>  
# 重定向，将输出的文件输出到另一个位置
# > 覆盖  >> 追加
```

例如，一个简单的C程序，`hello.c`

```c
#include "stdio.h"

int main(){
        printf("hello world\n");
        return 0;
}
```

编译一下（装一下gcc）

```
gcc hello.c
```

编译没有报错，就生成一个`.out`文件，运行一下

```shell
[root@localhost local]# ./a.out 
hello world
```

有时候我们并不希望这个数据输出到终端，所以我们可以使用重定向，将它写入到文件内（即使没有这个文件，他也会自动创建）

```shell
./a.out > b.txt
```

这样，我们就可以在`b.txt`内看到执行结果

### 管道

```shell
|
```

将上一个输出，转换为下一个的输入

例如

```shell
[root@localhost local]# ./a.out | cat | cat | cat |cat
hello world
```

俄罗斯套娃

### 日志查看

```shell
tail xxx [-n20]
# 查看文件的末尾数据，默认查看后10行
# -n 参数，指定显示几行
# -F 监听文件，文件一有变化，就立刻显示出
```

示例

```shell
tail -F b.txt
# 回车后，会显示末尾10行并且进入等待
# 使用另一个终端更改文件数据，保存后，会在这个终端立刻显示
```

### 后台运行

```shell
Ctrl + z 
# 切换到前台，程序后台运行
nohup java -jar xxx.jar & > a.log
# 后台运行xxx.jar，并且将日志输出到a.log
```