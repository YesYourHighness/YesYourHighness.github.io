---
title: Docker
date: 2021-11-16 20:30:36
tags: 
- Docker
categories: 
- Docker

---

<center>
引言：入门Docker，理解镜像、容器、仓库；学会使用基本的Docker命令！</center>

<!--more-->

# Docker入门

## 走进Docker

### 有关Docker的几个问题

> 1、什么是Docker？

一句话阐述：Docker是基于Go的开源**容器**项目

换句话说，**什么是容器？**

- 可以将Docker容器理解为一种**轻量级的虚拟机**
- 每个容器内运行着一个应用，不同的容器相互隔离，容器之间也可以通过网络互相通信。

很类似于一个虚拟机，但是开销远远低于传统的虚拟机，而且启动速度非常快

> 2、Docker有什么作用？

**解耦应用程序与运行平台**，可以保证**快速的分发与部署**

> 3、Docker与虚拟机的区别

| 特性     | Docker容器 | 虚拟机   |
| -------- | ---------- | -------- |
| 启动速度 | 秒级       | 分钟级   |
| 内存使用 | 很少       | 较多     |
| 隔离性   | 安全隔离   | 完全隔离 |
| 迁移性   | 优秀       | 一般     |

可见，除了隔离性略逊虚拟机一筹（并不是说Docker不安全），其他全部要优于虚拟机。

> 4、Docker容器与传统虚拟化技术的区别

Docker技术是**操作系统级别的虚拟化**

（不需要额外的虚拟机管理程序，内核级的虚拟化）

![Docker和传统的虚拟化方式的不同之处](http://img.yesmylord.cn//image-20211116165102160.png)

### Docker三大核心概念

> 镜像Image

可以理解为一个**只读的模板**

（一个镜像包含我们需要运行程序的必要环境）

> 容器Container

轻量的虚拟机，**Docker利用容器来隔离的运行应用**

- 容器是**从镜像创建的应用运行实例**
- 容器之间相互隔离

注意：镜像本身是**只读**的，容器从镜像启动的时候，会在镜像的最上层创建一个**可写层**

> 仓库Repository

类似于代码仓库（Github、码云），存放镜像的场所

- 注册服务器是存放仓库的地方（比如一个注册服务器下，会有Ubantu的仓库、CentoOs的仓库等等）

## Docker安装

环境：Centos 7

1. 升级`yum`环境

```
yum update
```

2. 安装docker需要的软件包

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

3. 设置yum源

```shell
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

4. 安装Docker（默认安装最新版）

```shell
sudo yum install docker-ce
# 直接安装最新版
```

（如果想安装其他版本）

```shell
yum list docker-ce --showduplicates | sort -r
# 查看仓库所有的docker版本
sudo yum install <FQPN> 
# 选择一个版本安装
```

5. docker安装后，查看是否安装成功

```shell
docker version
```

6. 启动Docker

```shell
# 启动Docker
systemctl start docker
# 重启
systemctl restart docker
# 关闭
systemctl stop docker
```

## 镜像的使用

### 获取镜像

我们可以去仓库下载镜像（类似于Git的操作）

```
docker pull name:[tag]
```

- `name`即你要下载的应用名称

- `tag`为标签，如果不选择的话，默认是`lastest`，也就是最新版本的（所以我们一定要指定标签）

- 严格来说，我们没有指定仓库，所以默认会从`registry.hub.docker.com`下载

  即此命令相当于`docker pull registry.hub.docker.com/name:[tag]`

- 如果需要安装非官方仓库的镜像的话，那么需要加上仓库名

---

这里使用该命令下载`nginx`作为一个示例：

```shell
[root@slave1 ~]# docker pull nginx:1.20.1
1.20.1: Pulling from library/nginx
b380bbd43752: Already exists 
83acae5e2daa: Pull complete 
33715b419f9b: Pull complete 
eb08b4d557d8: Pull complete 
74d5bdecd955: Pull complete 
0820d7f25141: Pull complete 
Digest: sha256:a98c2360dcfe44e9987ed09d59421bb654cb6c4abe50a92ec9c912f252461483
Status: Downloaded newer image for nginx:1.20.1
docker.io/library/nginx:1.20.1
```

可以看到，镜像文件由很多**层`layer`**构成，前面的一串`b380bbd43752`表示这一层的唯一ID

当**不同的镜像包括相同的层时，本地仅存储层的一份内容**，减小了需要的存储空间。

### 查看镜像

查看镜像的命令有如下几个：

1. 查看下载的镜像

```
docker images
-a 显示所有镜像，包括临时镜像
--no-trunc 默认情况下会截断显示内容，比如image id就截断显示了，此参数可以完全显示
-q 仅输出ID信息
```

此处做示例

```shell
[root@slave1 ~]# docker images --no-trunc
REPOSITORY   TAG       IMAGE ID                                                                  CREATED       SIZE
mysql        5.7       sha256:938b57d64674c4a123bf8bed384e5e057be77db934303b3023d9be331398b761   4 weeks ago   448MB
nginx        1.20.1    sha256:c8d03f6b8b915209c54fc8ead682f7a5709d11226f6b81185850199f18b277a2   5 weeks ago   133MB
```

可见，`Image Id`是很长的串，只不过可以用前几位代替完整的ID

2. 可以给镜像添加自定义的标签

命令：

```
docker tag name:[tag] my-tag
```

- `my-tag`：表示自己可以自定义的标签

下面是一个示例

```shell
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
nginx        1.20.1    c8d03f6b8b91   5 weeks ago   133MB
[root@slave1 ~]# docker tag nginx:1.20.1 my-nginx
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
my-nginx     latest    c8d03f6b8b91   5 weeks ago   133MB
nginx        1.20.1    c8d03f6b8b91   5 weeks ago   133MB
```

可以看到，`my-nginx`与`nginx`两个的ID值一样，说明两个指向同一个镜像

我们自定义的标签只是一个软连接而已

3. 查看镜像详细信息

```
docker inspect name:tag
```

会返回一个json串，如果我们只想查看其中的部分数据可以加`-f`参数，例如这样（记着加`.`）

```
docker inspect my-nginx -f {{".Metadata"}}
```

### 搜索镜像

```
docker search name
-f 可以指定额外的参数
-f stars=100 显示star数大于100的镜像
-f is-automated=true 显示可以自动装配的镜像
```

示例：显示所有stars大于100的并且可以自动装配的nginx镜像

> 自动装配的意思是，可以允许用户通过Docker Hub指定跟踪一个目标网站（目前支持GitHub或BitBucket）上的项目，一旦项目发生新的提交，则自动执行创建

```sh
[root@slave1 ~]# docker search -f stars=100 -f is-automated=true nginx
NAME                          DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
jwilder/nginx-proxy           Automated Nginx reverse proxy for docker con…   2094                 [OK]
richarvey/nginx-php-fpm       Container running Nginx + PHP-FPM capable of…   818                  [OK]
tiangolo/nginx-rtmp           Docker image with Nginx using the nginx-rtmp…   145                  [OK]
jlesage/nginx-proxy-manager   Docker container for Nginx Proxy Manager        143                  [OK]
alfg/nginx-rtmp               NGINX, nginx-rtmp-module and FFmpeg from sou…   110                  [OK]
```

### 删除镜像

```
docker rmi tag|id
```

- 如果参数指定为一个`tag`，那么此命令只会删除该镜像的一个标签而已（如果这个镜像只有一个标签，那么会被立即删除）
- 参数指定为`image id`的话，会先删除该镜像的所有标签，然后删除该镜像本身

示例：

```sh
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
my-nginx     latest    c8d03f6b8b91   5 weeks ago   133MB
nginx        1.20.1    c8d03f6b8b91   5 weeks ago   133MB
[root@slave1 ~]# docker rmi my-nginx
Untagged: my-nginx:latest // 删除了标签
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
nginx        1.20.1    c8d03f6b8b91   5 weeks ago   133MB
[root@slave1 ~]# docker rmi nginx:1.20.1
Untagged: nginx:1.20.1
Untagged: nginx@sha256:a98c2360dcfe44e9987ed09d59421bb654cb6c4abe50a92ec9c912f252461483
Deleted: sha256:67a7407724b6c71e2355fc2236b5be27d1f03bf9cbdffdfbb97c1d2a326ccf94
...// 真正删除了所有的镜像
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
```

## 容器的使用

### 创建、运行容器

1. 创建容器（基于一个镜像，创建一个容器）

```
docker create [参数] name:tag
# 会返回一个容器的ID Container ID
```

2. 启动容器

```
docker start id
# 此处的id只需要输入一部分即可
```

创建并启动（建议直接使用`run`命令，等同于上述创建并启动容器）

3. 创建并启动容器

```shell
docker run [options] image
```

其中`options`可选的参数很多，下面介绍几个常用的

```shell
-i # 让容器的标准输入保持打开
-t # 分配一个伪终端并绑定到容器的标准输出上
-d # 后台运行（守护态运行）
-P # 容器内随机映射一个端口
-p # 指定端口映射，比如 -p 3306:3306 就指定主机的3306端口与容器的3306端口进行映射，此时访问主机的3306端口，相当于访问容器的3306端口
--name # 指定此容器的名字
```



### `run`命令的执行内容

当利用`docker run`来创建并启动容器时，Docker在后台运行的标准操作包括：

1. 检查本地是否存在指定的镜像，**不存在就从公有仓库下载**；
2. 利用镜像**创建**一个容器，并**启动**该容器；
3. **分配一个文件系统**给容器，并**在只读的镜像层外面挂载一层可读写层**；
4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中；
5. 从网桥的地址池配置一个IP地址给容器；
6. 执行用户指定的应用程序；
7. 执行完毕后容器被自动终止。

### 查看容器

1. 查看所有的容器

```
docker ps
-a 可以显示所有的容器，包括没有在运行的容器
```

2. 查看容器的日志

```
docker logs cotainerId
```

### 进入容器

1. 进入容器

```
docker attach
```

2. 执行命令

```
docker exec
```

### 终止、重启、删除容器

1. 终止容器

首先向容器发送SIGTERM信号，**等待一段超时时间**（默认为10秒）后，再发送SIGKILL信号来终止容器

```
docker stop id
-t 可以指定终止的时间
```

2. 对于已经终止的容器，需要使用`docker start`来启动
3. 如果想要重启一个容器

```
docker restart id
```

4. 强制终止一个容器

会直接发送SIGKILL信号

```
docker kill id
```

5. 删除容器

只能删除非运行态的容器！（不建议使用`-f`参数强制删除）

```
docker rm id
```

## 参考资料

- Docker技术入门与实战



