---
title: Hadoop安装与集群配置
date: 2021-03-21 21:25:43
tags: 
- 大数据
- hadoop
categories: 
- hadoop
---

 <center>
引言：

hadoop的安装与集群配置

</center>

<!--more-->

# Hadoop安装与集群配置

## 安装Hadoop

首先下载Hadoop，本篇使用hadoop2.7.3



### 安装前提

#### JDK环境

安装JDK

```
# 安装Java
yum install -y java-1.8.0-openjdk.x86_64
```

JDK环境在1.6以上，并在`/etc/profile`中配置环境变量

```shell
# JavaHome，这里的路径根据自己的情况定
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.el7_9.x86_64/jre
80 export PATH=$PATH:$JAVA_HOME/bin
```

**注意：**如果没有`jps`命令，是没有`openjdk-devel`这个包

```
yum install java-1.8.0-openjdk-devel.x86_64
```



#### 虚拟机准备

linux操作系统，ubuntu、centos都可以，这里创建了三个Centos7的虚拟机。

每一个虚拟机给了10G的磁盘空间、其他默认，其中一台机器安装了图形界面，剩下的两个虚拟机最小安装，给三台虚拟机改名字，修改`/etc/hostname`文件

```shell
# 这里起名为
# 虚拟机1的hostname文件
master
# 虚拟机2的hostname文件
slave1
# 虚拟机3的hostname文件
slave2
```



### 解压并配置

1. 解压（这里解压到了`/usr/local/hadoop`）

   ```shell
   # 解压 ，可以加参数-C指定解压目录
   tar -zxvf hadoop-2.7.3.tar.gz
   ```

2. 配置`hadoop/etc/hadoop/hadoop-env.sh`中的Java环境为**当前的Java环境**（Java要求1.6以上）

3. 检测，进入`hadoop/bin/`下

   ```
   ./hadoop version
   ```

   打印出hadoop版本信息即安装成功，打印的信息如下：

   ```
   Hadoop 2.7.3
   Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r baa91f7c6bc9cb92be5982de4719c1c8af91ccff
   Compiled by root on 2016-08-18T01:41Z
   Compiled with protoc 2.5.0
   From source with checksum 2e4ce5f957ea4db193bce3734ff29ff4
   This command was run using /usr/local/hadoop/share/hadoop/common/hadoop-common-2.7.3.jar
   ```
   
   如果这一步报错，请检查Java环境变量是否正确

4. 将hadoop的命令配置环境变量，修改`/etc/profile`文件，添加如下

   ```shell
   # HadoopHome，根据自己安装的hadoop位置进行配置
   export HADOOP_HOME=/usr/local/hadoop
   export PATH=$PATH:$HADOOP_HOME/bin
   export PATH=$PATH:$HADOOP_HOME/sbin
   ```

### Hadoop目录简介

```shell
bin		#hadoop的命令
sbin 	 #hdfs启动的命令
logs 	 #日志文件，namenode与datanode的日志文件都在这里
etc 	 #配置文件，下文会对这里的文件进行配置
```



## Hadoop伪分布式安装

伪分布式：即在一台机器上搭建

### SSH无密码登录

hadoop不支持密码登录，所以我们要先使用SSH进行无密码验证

```shell
# 1. 首先生成SSH秘钥
ssh-keygen -t rsa
# 2. 其次将公钥追加写入到authorized_keys文件内
#	（authorized_keys文件会自动创建）
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

检测一下

```
ssh localhost
```

如果不需要密码，说明配置正确



### etc文件配置

修改`etc/hadoop/`下的文件

- `core-site.xml`文件

  ```xml
  <configuration>
          <property>
              <!-- 配置存放临时数据的目录，file代表本地目录 -->
                  <name>hadoop.tmp.dir</name>
                  <value>file:/usr/local/hadoop/tmp</value>
          </property>
          <property>
              <!-- fs.defaultFS表示HDFS的访问地址 -->
                  <name>fs.defaultFS</name>
                  <value>hdfs://localhost:9000</value>
          </property>
  </configuration>
  ```

- `hdfs-xml.xml`配置

  ```xml
  <configuration>
          <property>
              <!-- 配置副本的数量，由于是伪分布式，所以这里仅能配 1 -->
                  <name>dfs.replication</name>
                  <value>1</value>
          </property>
          <property>
              <!-- 配置名称节点的元数据的保存目录 -->
                  <name>dfs.namenode.name.dir</name>
                  <value>file:/usr/local/hadoop/tmp/dfs/name</value>
          </property>
          <property>
              <!-- 配置数据结点的数据保存目录 -->
                  <name>dfs.datanode.data.dir</name>
                  <value>file:/usr/local/hadoop/tmp/dfs/data</value>
          </property>
  </configuration>
  ```

### 格式化namenode

执行命令

```shell
hadoop namenode -format
```

（如果没有配置hadoop环境变量，那么去`bin`下执行这行命令）

执行这条命令后，我们会发现多了一个目录`tmp`



### 启动hadoop

执行命令

```shell
start-all.sh
```

（如果没有配置hadoop环境变量，那么去`sbin`下执行这行命令）

访问`localhost:50070`查看即可



## Hadoop集群安装

注意：

1. 三台虚拟机都得有java并配置环境变量
2. slave虚拟机先不必安装hadoop

### 网络配置

首先确保三个虚拟机在同一个网段内，然后配置master虚拟机的`/etc/hosts`文件

使用`ip addr`或者`ifconfig`查看对应虚拟机的ip地址

向文件内添加如下

```shell
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
 # 以下是添加的内容
192.168.235.143 master
192.168.235.144 slave1
192.168.235.145 slave2
```

使用ping命令查看master是否可以联系到slave1、slave2

```
ping slave1

ping slave2
```



### SSH无密码登录

1. Master进行第二章的SSH配置后即可，不用再改动

2. 将Master的**公钥**分别复制一份到对应slave1、slave2虚拟机的`~/.ssh/authorized_keys`文件内（没有文件则创建对应文件）

   可以使用scp命令传输文件

   ```shell
   # 注意，首先要在对应虚拟机下创建对应的路径
   # scp 文件 虚拟机名称:路径
   scp id_rsa.pub slave1:~/.ssh/authorized_keys
   scp id_rsa.pub slave2:~/.ssh/authorized_keys
   ```

3. 检查配置是否成功

   ```
   ssh slave1
   ```

   ![image-20210321205359202](http://img.yesmylord.cn//image-20210321205359202.png)

   使用`exit`可以退回原主机

### 配置文件

修改`etc/hadoop/`下的文件

- `core-site.xml`文件

  ```xml
  <configuration>
          <property>
              <!-- 配置存放临时数据的目录，file代表本地目录 -->
                  <name>hadoop.tmp.dir</name>
                  <value>file:/usr/local/hadoop/tmp</value>
          </property>
          <property>
              <!-- fs.defaultFS表示HDFS的访问地址 -->
                  <name>fs.defaultFS</name>
                  <value>hdfs://master:9000</value>
          </property>
  </configuration>
  ```

- `hdfs-xml.xml`配置

  ```xml
  <configuration>
           <property>
              <!-- 配置第二名称节点的管理端口 -->
                  <name>dfs.namenode.secondary.http-address</name>
                  <value>Master:50090</value>
          </property>
          <property>
              <!-- 配置副本的数量，这里配置为3 -->
                  <name>dfs.replication</name>
                  <value>3</value>
          </property>
          <property>
              <!-- 配置名称节点的元数据的保存目录 -->
                  <name>dfs.namenode.name.dir</name>
                  <value>file:/usr/local/hadoop/tmp/dfs/name</value>
          </property>
          <property>
              <!-- 配置数据结点的数据保存目录 -->
                  <name>dfs.datanode.data.dir</name>
                  <value>file:/usr/local/hadoop/tmp/dfs/data</value>
          </property>
  </configuration>
  ```

- `mapred-site.xml`配置（注意这个文件需要复制`mapred-site.xml.template`并改名）

  ```xml
  <configuration>
          <property>
                  <name>mapreduce.framework.name</name>
                  <value>yarn</value>
          </property>
          <property>
                  <name>mapreduce.jobhistory.address</name>
                  <value>master:10020</value>
          </property>
          <property>
                  <name>mapreduce.jobhistory.webapp.address</name>
                  <value>master:19888</value>
          </property>
  </configuration>
  ```

- 配置`yarn-site.xml`

  ```xml
  <configuration>
  <!-- Site specific YARN configuration properties -->
          <property>
                  <name>yarn.resourcemanager.hostname</name>
                  <value>master</value>
          </property>
          <property>
                  <name>yarn.nodemanager.aux-services</name>
                  <value>mapreduce_shuffle</value>
          </property>
  </configuration>
  ```



### 发送hadoop整个文件给slave虚拟机

先将整个hadoop文件打包

```shell
# tar -zcf 放置的位置 打包的文件或文件夹
tar -zcf ~/hadoop.tar.gz hadoop/
```

发送给两个虚拟机

```shell
scp hadoop.tar.gz slave1:/usr/local/
scp hadoop.tar.gz slave1:/usr/local/
```

去对应虚拟机下解压即可



### master配置slaves文件

在`/etc/hadoop/`目录下有一个`slaves`文件

这个文件内，默认是`localhost`，如果我们master不仅仅想做`namenode`，还想做`datanode`的话，可以留着

这里我改成了

```
slave1
slave2
```



### 启动hadoop

执行命令

```shell
start-all.sh
```

（如果没有配置hadoop环境变量，那么去`sbin`下执行这行命令）



master虚拟机使用`jps`查看进程：（左边的数字是进程号）

```
[root@master ~]# jps
71842 SecondaryNameNode
71671 NameNode
72007 ResourceManager
73961 Jps
```

slave虚拟机查看

```
[root@slave1 hadoop]# jps
4928 DataNode
5026 NodeManager
5496 Jps
```

访问`localhost:50070`查看





下滑可以看到我们存活的节点是2个

![image-20210321211416324](http://img.yesmylord.cn//image-20210321211416324.png)

点击可以查看详细情况

![image-20210321211440474](http://img.yesmylord.cn//image-20210321211440474.png)

这里就完成了



**注意：**如果显示的节点是0的话，是防火墙的问题，关掉防火墙即可

```
systemctl stop firewalld.service
```





## 报错处理

总共遇到三个问题

1. jps命令找不到

   解决：安装`openjdk-devel`这个库

   ```
   yum install java-1.8.0-openjdk-devel.x86_64
   ```

2. 配置成功后，打开50070端口web应用，显示启动的datanode节点是0

   解决：关闭防火墙

   ```
   systemctl stop firewalld.service
   ```