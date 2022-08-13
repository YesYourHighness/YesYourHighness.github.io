---
title: 编译运行nginx
date: 2020-07-02 14:20:19
tags:
- nginx
categories: 
- 编译
---

<center>
引言： 

使用linux编译运行nginx！！

</center>

<!-- more -->

# 编译运行nginx

## 安装环境

CentOS7安装编译nginx1.19.0

## 步骤：

**1. Nginx官网**

在[Nginx](http://nginx.org/en/download.html)官网下载安装包 nginx-1.19.0

[官网有详细的安装部署说明](http://nginx.org/en/docs/)

----

**2. CentOS7中解压**

```shell
tar -zxvf nginx-1.19.0.tar.gz
```

进入解压后的文件，会看到

有多个文件，我们主要会用到的有`conf` `configure`

-----

**3. configure**

切换到nginx目录内，执行

```shell
./configure --prefix=/usr/local/demo
```

其他参数官网十分详细，这里列举几个重要的配置

```shell
--prefix=path
# 定义保存服务器文件的目录，默认为 /usr/local/nginx

--with-http_ssl_module
# 将支持https协议
```

**4. make**

依然在原目录下执行

```shell
make
```

编译安装

**5. make install**

执行安装，安装后`--prefix`指定的安装目录下会出现

![image-20200702135045948](http://img.yesmylord.cn/image-20200702135045948.png)

如此的目录。

`conf`下就是配置文件，我们待会儿需要对其目录下的`nginx.conf`进行配置

`sbin`下是执行命令的目录

**6. 找一个博客作为检验**

我把博客放在了`/usr/nginx/blog`内

![img](http://img.yesmylord.cn/image-20200702135813487.png)

博客目录下有`html`文件以及静态图片文件夹还有css文件

---

**7. 配置conf**

在`--prefix`后的路径下，进入conf目录，配置`nginx.conf`文件，将

```properties
http{
server{
	listen 80;
	# 监听的端口地址
	server_name localhost;
	
	location /{
		root html;
		index index.html index.htm
	}
	# 把这里的root 配置到blog的目录
}
}
```

---

**8. 启动配置**

这里先学习一下`nginx`的命令

> ```shell
> nginx -s signal
> ```

Where *signal* may be one of the following:

```
stop — fast shutdown

quit — graceful shutdown

reload — reloading the configuration file

reopen — reopening the log files
```

优雅的杀死nginx

```shell
kill -s QUIT [nginx的pid]
```

学会了这些，进入`sbin`目录执行

```shell
./nginx
```

就启动了，打开浏览器，输入`localhost`，发现已经能愉快的访问了

![image](http://img.yesmylord.cn/image-20200702141149556.png)

----

# 遇到的小bug

可能会出现这种情况

![img](http://img.yesmylord.cn/image-20200702141508419.png)

有可能是你执行了两遍`./nginx`，80端口被占用，就会报错

解决方法：

```shell
netstat -ntlp
# 查看端口为80的哪一个进程
kill -s QUIT [nginx的pid]
# 使用官方手法，优雅的杀死nginx
```

ENDING