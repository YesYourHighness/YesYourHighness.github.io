---
title: hadoop基本运行与源码编译
date: 2020-07-04 09:35:02
tags:
- hadoop
categories: 
- 编译
---

<center>
引言： 

hadoop的基本使用与源码编译

</center>

<!-- more -->

# Hadoop启动及其基本操作

## 前置需求

1. JDK（官方要求不同版本使用不同的JDK，由于我们使用2.10.0版本hadoop，必须使用JDK7及以上，JDK太高又会有warning，使用JDK8是最好的选择）
2. ssh及pdsh管理ssh资源

```
rpm -qa|grep -E "openssh"
# 检查是否安装ssh
echo $JAVA_HOME
# 查看javahome路径
```

3. 下载Hadoop。[官网链接](https://hadoop.apache.org/releases.html)，千万注意，下`hadoop-2.10.0-src.tar.gz`，别下成原码了！！

## 安装Hadoop及实现伪分布式配置

1. [官网链接](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)：详细配置及参考

2. 将下载的Hadoop压缩包随意的解压在一个包内![img](http://qcrt74khz.bkt.clouddn.com/image-20200703101611210.png)

   这就是解压后的文件

3. 进入etc目录下，配置`hadoop-env.sh`

   ````shell
   # The java implementation to use.
   export JAVA_HOME=[配置到JAVA_HOME]
   # 使用 echo $JAVA_HOME 查看JAVA_HOME路径
   ````

4. 配置`core-site.xml`及`hdfs-site.xml`

   ```shell
   # core-site.xml
   <configuration>
       <property>
           <name>fs.defaultFS</name>
           <value>hdfs://localhost:9000</value>
       </property>
   </configuration>
   
   --------------------------------
   
   # hdfs-site.xml
   <configuration>
       <property>
           <name>dfs.replication</name>
           <value>1</value>
       </property>
   </configuration>
   ```

   ----------

   官网配置了这些就可以启动了，下面是百度到的其他配置，便于学习

   ```xml
   <!--core-site.xml-->
   <configuration>
           <!-- 指定HDFS的nameservice -->
           <property>
                   <name>fs.defaultFS</name>
                   <value>hdfs://zachary-pc:9000</value>
           </property>
   		<!-- 缓冲区大小 -->
           <property>
                   <name>io.file.buffer.size</name>
                   <value>131072</value>
           </property>
           <!-- 指定临时文件目录 -->
           <property>
                   <name>hadoop.tmp.dir</name>
                   <value>/home/zachary/hadoop-2.10.0/tmp</value>
           </property>
   </configuration>
   
   
   -------------------
   <!--hdfs-site.xml-->
   <configuration>
   <!-- configuration for NameNode:-->
           <property>
                   <name>dfs.namenode.name.dir</name>
                   <value>/home/zachary/hadoop-2.10.0/name</value>
           </property>
           <!-- 块大小 -->
           <property>
                   <name>dfs.blocksize</name>
                   <value>268435456</value>
           </property>
           <property>
                   <name>dfs.namenode.handler.count</name>
                   <value>100</value>
           </property>
   <!-- configuration for DateNode:-->
           <property>
                   <name>dfs.datanode.data.dir</name>
                   <value>/home/zachary/hadoop-2.10.0/data</value>
           </property>
           <!-- 指定HDFS副本的数量 -->
           <property>
                   <name>dfs.replication</name>
                   <value>1</value>
           </property>
   </configuration>
   
   ------------
   <!--mapred-site.xml-->
   <configuration>
           <property>
                   <name>mapreduce.framework.name</name>
                   <value>yarn</value>
           </property>
   </configuration>
   
   ------------
   <!--mapred-site.xml-->
   <configuration>
   <!-- Site specific YARN configuration properties -->
           <!-- configuration for ResourceManger: -->
           <property>
                   <name>yarn.resourcemanager.hostname</name>
                   <value>zahcary-pc</value>
           </property>
           <!-- configuration of NodeManager: reducer获取数据方式 -->
           <property>
                   <name>yarn.nodemanager.aux-services</name>
                   <value>mapreduce_shuffle</value>
           </property>
         <!-- 配置外网只需要替换外网ip为真实ip，否则默认为 localhost:8088 -->
   	  <!-- <property>
   	          <name>yarn.resourcemanager.webapp.address</name>
   	          <value>外网ip:8088</value>
   	  </property> -->
   </configuration>
   
   ```

   

5. 配置SSH免密登录

   在hadoop根目录下，输入

   ```shell
   # 生成公钥和秘钥，他们生成后是家目录(~)下的id_rsa和id_rsa.pub前者为私钥，后者为公钥
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   chmod 0600 ~/.ssh/authorized_keys
   ```

6. 格式化`namenode`

   ```shell
   bin/hdfs namenode -format
   ```

7. 启动守护程序

   ```shell
   sbin/start-dfs.sh
   ```

8. 启动hadoop，在`sbin`目录下

   ```shell
   ./start-all.sh
   # 启动
   ./stop-all.sh
   # 关闭
   ```

9. 输入`jsp`，如果有6个进程出现，说明启动成功

10. 访问端口50070就是HDFS管理界面

    ![image-20200703103649720](http://qcrt74khz.bkt.clouddn.com/image-20200703103649720.png)

    查看端口8088，结点管理页面

    ![image-20200703103826728](http://qcrt74khz.bkt.clouddn.com/image-20200703103826728.png)

## Hadoop命令行基本的操作

```shell
bin/hdfs dfs -mkdir /user/t
# 创建指定目录

hadoop fs –rmr /user/t
# 删除指定的文件夹

hadoop fs -touchz /user/new.txt
# 在hadoop指定目录下新建空文件

hadoop dfs –ls [文件目录]
# 查看

hadoop fs –put [本地地址] [hadoop目录]
hadoop fs –put /home/t/file.txt  /user/t   
# 上传本地文件

hadoop fs –put [本地目录] [hadoop目录] 
hadoop fs –put /home/t/dir_name /user/t
# 上传本地文件夹(dir_name是文件夹名)

hadoop fs -get [文件目录] [本地目录]
hadoop fs –get /user/t/ok.txt /home/t
# 将hadoop上某个文件down至本地已有目录下

hadoop fs –rm [文件地址]
hadoop fs –rm /user/t/ok.txt
# 删除hadoop上指定文件

hadoop fs –rm [目录地址]
hadoop fs –rmr /user/t
# 删除hadoop上指定文件夹（包含子目录等）

hadoop fs –mv /user/test.txt /user/ok.txt
# 将hadoop上某个文件重命名（将test.txt重命名为ok.txt）

hadoop job –kill  [job-id]
# 将正在运行的hadoop作业kill掉

```

# Hadoop 源码编译

## 环境要求

- 要求`apache-ant`
- 要求`jdk1.7`（亲自尝试，`jdk1.8`在最后会报出一大片红色Error）
- 要求`maven`（maven要求在3.3.0以上，最好选择最新版本的maven）
- 要求剩余磁盘大于两个G（`df -h`查看剩余磁盘容量）
- 编译`hadoop-2.10.0-src.tar.gz`

## 编译过程

### Maven配置

1. 解压软件到一个位置。
2. 在`/etc/profile`中配置环境路径
3. 使用`mvn -version`查看版本信息

Tips：如果下载太慢的话，可以中断后，执行`mvn -clean`然后换成阿里的镜像，重新下载

### ANT配置

1. 同Maven的安装的配置，解压缩
2. 在`/etc/profile`中配置环境路径
3. 使用`ant -version`查看版本信息

### 安装`glibc-headers`和`g++`

```shell
sudo yum install glibc-headers
# 安装glibc-headers
sudo yum install gcc-c++
# 安装g++
sudo yum install make
# 安装make
sudo yum install cmake
# 安装cmake
```



### 安装配置protobuf

1. 解压文件
2. 进入到`protobuf-2.5.0`目录下
3. `./configure`命令
4. `make`命令
5. `make check`命令
6. `make install`命令
7. `ldconfig`命令
8. 配置环境变量

### 环境变量配置如图

![image-20200704091051391](http://qcrt74khz.bkt.clouddn.com/image-20200704091051391.png)

注意！！**protobuf没有bin，不需要配置到bin下**

**如果配置环境后，查看版本没有成功，就输入`source /etc/profile`重编一下path**

### 安装其他环境

```shell
sudo yum install openssl-devel
# 安装openssl-devel库
sudo yum install ncurses-devel
# 安装ncurces-devel库
```

### 开始编译

1. 解压`hadoop`源码包到指定目录

2. 进入该目录，使用mvn执行编译（这个过程十分漫长，请耐心等待）

3. `df -h`查看磁盘

   ![sdjkal](http://qcrt74khz.bkt.clouddn.com/sdjkal.jpg)

   ```shell
   # 注意！！！查看磁盘是否足够，是否有2g以上，然后再执行命令（在漫长的等待后，我因为磁盘不足，failed一次，血的教训）
   mvn package -Pdist,native -DskipTests -Dtar
   ```

4. 如果你的版本和我要求的版本都一样的话（要求jdk1.7、Maven3.3.0+、ant无要求，protobuf2.5）是可以成功的

![img](http://qcrt74khz.bkt.clouddn.com/hadoop)

4. 成功