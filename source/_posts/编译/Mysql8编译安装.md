---
title: Mysql8编译安装
date: 2020-07-07 10:18:49
tags:
- mysql
categories: 
- 编译
---


<center>
引言： 

centOs 编译运行mysql

</center>

<!-- more -->

# MySql8.0.20编译安装



## 前置要求：

### 更新cmake

> cmake要求3.0及以上

```shell
yum -y remove cmake
# 如果你曾yum安装过cmake，就卸载一下吧
wget https://github.com/Kitware/CMake/releases/download/v3.15.3/cmake-3.15.3-Linux-x86_64.tar.gz
# 下载压缩包
tar -zxvf cmake-3.15.3-Linux-x86_64.tar.gz
# 设置环境变量
vim /etc/profile

export CMAKE_PATH=/usr/local/cmake
export PATH=$CMAKE_PATH/bin:$PATH

# 更新path
source /etc/profile
# 查看cmake信息
cmake -v
```



### 更新gcc

>  gcc 要求5.4及以上

```shell
sudo yum install centos-release-scl
sudo yum install devtoolset-7-gcc*
scl enable devtoolset-7 bash
gcc --version
```

更新完成后，我们需要改动`/usr/bin/`下的文件

```shell
# 先删除这个文件夹下的 cc 和 c++两个文件
# 建立链接
sudo ln -s [gcc-c++路径] /usr/bin/c++
# 建立链接
sudo ln -s [gcc路径] /usr/bin/cc
# 可以使用以下命令查看路径
which gcc
```



### boost文件

boost文件我们可以直接下载，也可以在命令行直接下载。这里我们直接在后面命令行中下载即可



## 安装Mysql

1. 解压mysql文件后，进入文件目录

为了避免每次失败后，得删除整个mysql文件重新解压，我们建一个`build`文件夹

```shell
mkdir build
cd build
cmake .. -DDOWNLOAD_BOOST=1 -DWITH_BOOST=[配置一个下载的目的路径]  -DFORCE_INSOURCE_BUILD=1
```

以下是百度到的更多的配置，记录一下：

```shell
sudo cmake .. \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_unicode_ci \ #
-DENABLED_LOCAL_INFILE=ON \
-DWITH_SSL=system \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql/server \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DMYSQL_TCP_PORT=3306 \
-DDOWNLOAD_BOOST=0 \
-DWITH_BOOST=/usr/mysql-boost/boost_1_70_0.tar.gz
```

- `DDEFAULT_CHARSET`：指定默认字符集为`utf8mb4`，因为历史遗留问题，MySQL中的utf8不是真正的utf8，而是阉割版的，最长只有三个字节，当遇到四个字节的utf8编码时，会导致存储异常。从5.5.3开始，使用utf8mb4实现完整的utf8。
- `DDEFAULT_COLLATION`：排序规则，默认为utf8mb4_0900_ai_ci，属于utf8mb4_unicode_ci的一种。0900指的是Unicode校对算法版本，ai是指口音不敏感（as表示敏感），ci指不区分大小写（cs表示区分）。utf8mb4_unicode_ci表示基于标准的的Unicode来排序和比较，能够在各种语言之间精确排序，而utf8mb4_general_ci遇到某些特殊的字符集时排序结果可能不一致，准确性较差，但是性能较好，比较和排序时候更快。
- `DENABLED_LOCAL_INFILE`表示能否使用load data命令。
- `DWITH_SSL`表示使用系统的SSL库，若不使用系统的请自定义路径。
- `DCMAKE_INSTALL_PREFIX`：MySQL安装目录。
- `DMYSQL_DATADIR`：MySQL数据目录，初始时为空。
- `DMYSQL_TCP_PORT`：端口，默认3306。
- `DDOWNLOAD_BOOST`：取值0或1，是否下载Boost库。
- `DWITH_BOOST`：若不下载Boost库的话，是本地Boost库的位置，若下载Boost表示下载位置

2. 然后在mysql根目录下就可以`make`了
3. 然后`make install`

## 配置mysql

这才是真正的难处啊！！！

摸爬滚打，一个坑一个坑菜，试出来：

1.  ```shell
   # 添加用户组
   groupadd mysql
   useradd -g mysql mysql
   chown -R mysql:mysql /usr/local/mysql
    ```

2. ```shell
   # 初始化mysql
   cd /usr/local/mysql/bin
   ./mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
   ```

3. ```shell
   echo "export PATH=$PATH:/usr/local/mysql/bin" >> /etc/profile
   source /etc/profile
   ```

4. ```shell
   # 配置my.cnf文件
   [mysqld]
   skip-grant-tables
   # 跳过密码
   basedir = /usr/local/mysql
   datadir = /usr/local/mysql/data
   socket = /usr/local/mysql/mysql.sock
   user = mysql
   symbolic-links=0
   [mysqld_safe]
   log-error = /usr/local/mysql/log-error/mariadb.log
   pid-file = /usr/local/mysql/mariadb.pid
   ```

5. ```
   # 创建日志授予权限
   mkdir -p /usr/local/mysql/log-error/
   touch /usr/local/mysql/log-error/mariadb.log
   chown -R mysql:mysql /usr/local/mysql/log-error/mariadb.log
   ```

6. 

   ```shell
   # 启动服务
   /usr/local/mysql/support-files/mysql.server start
   ```

7.  ```shell
   # bin下登录mysql
   ./mysql -u root 
    ```

   这里会报一个错误，

   ![image-20200705113857303](http://qcrt74khz.bkt.clouddn.com/image-20200705113857303.png)

   这里需要建立一个软链接

   查看`/etc/my.cnf`文件下的`socket`路径，建立它和`tmp/mysql.sock`的软链接

   ```shell
   ln -s /usr/local/mysql/mysql.sock tmp/mysql.sock
   ```

8. 重新启动

   ```
   ./mysql restart
   ```

   太难了！！启动六个小时没启动成功。最后成功了

### 启动成功

![image-20200705112843183](http://qcrt74khz.bkt.clouddn.com/image-20200705112843183.png)