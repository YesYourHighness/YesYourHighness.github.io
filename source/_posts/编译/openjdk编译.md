---
title: openjdk编译
date: 2020-07-01 12:02:53
tags: 
- openjdk
categories: 
- 编译
---

<center>
引言： 

使用linux编译jdk源码！！

</center>

<!-- more -->


# CentOS7 编译 OpenJDK

## 前述 ：

​       最开始是想编译JDK8的，但是无奈折腾了一下午都没有找见门路。决定曲线救国，先编译JDK12.

​       果然越新的JDK越容易构建，终于成功的编译。


## 步骤：

编译JDK12，需要如下准备：

1. 准备openjdk12的压缩文件，解压

   a) [官方网址](http://hg.openjdk.java.net/jdk) 下载 jdk 12

   b)  完成后XFTP丢入虚拟机，这里放在了`/usr/openjdk12`下

   c)  解压文件：`tar -zxvf xxx.tar.gz`  或者` unzip xxx.zip`

   d) 在DOC目录下，会有`Building.html`文件，这里有详细的配置需求，可以参考。

2. 准备jdk10或者jdk11，并且配置环境变量

   a)  亲爱的甲骨文公司下载jdk10或者11，这里使用了jdk11，解压到一个地方，这里解压到了 `/usr/local/java/` 目录下

   b)  配置`/etc/profiles`

   c)  在最底下加两行

   ``` shell
    export PATH=$JAVA_HOME/bin:$PATH
    export JAVA_HOME=/usr/local/install/jdk-11
   ```

   d)  更新配置 `source /etc/profile`

   e)  用`java -version`检查一下，是不是11（不是11，试着检查配置是否正确，路径是否正确，再更新一下）

3. 配置

   打开openjdk下的doc目录，会发现有一个 building.html 文件，这个文件中的 Build jdk requirements，要用yum装了这里所有的配置：

```shell
sudo yum install autoconf
sudo yum install freetype-devel
sudo yum install cups-devel
sudo yum install libXtst-devel libXt-devel libXrender-devel libXrandr-devel libXi-devel
sudo yum install alsa-lib-devel
sudo yum install libffi-devel
sudo yum install -y gcc gcc-c++
sudo yum install fontconfig-devel
```

还有其他的一些配置，在下一步的时候会报错，提示安装。

​    执行配置：

```shell
bash configure --with-debug-level=slowdebug --with-boot-jdk=[这里放引导jdk的路径] --disable-warnings-as-errors —with-version-string=12-internal+0-wenhaodong
```

​	重要配置参数详细设置：可以使用`bash configure --help`查看

```shell
--with-version-string=<string> - Specify the version string this build will be identified with.
# 可以配置自己的名字 —with-version-string=12-internal+0-wenhaodong
--with-boot-jdk=<path> - Set the path to the Boot JDK
# 配置boot-jdk路径（boot-jdk即引导jdk）
```



​    显示类似这样，就成功了！！

![image](http://qcrt74khz.bkt.clouddn.com/configure.png)

   

4. 编译

​        终于到了这一步，别紧张，先去看看虚拟机处理器和内存数，别太低，太低会很慢，会让你感觉是卡了，真的很难受，这里配置了 2 处理器 4G内存

输入 

```shell
make images 
```

make的详细参数：

```shell
make images # 构建jdk映像
make docs # 构建jdk文档
make all # 构建全部生成映像
make clean # 删除make生成的文件，但不会删除configure的文件
make dist-clean # 删除所有文件
```

然后去打水喝水，深呼吸祷告。

出现这样，就代表你成功了！！

![image](http://qcrt74khz.bkt.clouddn.com/make.png)

5. 成功！！

   在build目录下中有一个jdk文件，那就是我们编译出来的文件。

   自己编一个`.java`文件，用`jdk/bin`下的命令编译、运行 最后成功了

   令人感动

![image](http://qcrt74khz.bkt.clouddn.com/yunxing.png)



成功！！激动。