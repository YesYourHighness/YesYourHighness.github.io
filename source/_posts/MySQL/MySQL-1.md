---
title: MySQL-1
date: 2019-08-26 22:07:20
tags: 
- MySQL
categories: 
- 后台
- MySQL
---

<center>
引言：

终于步入了正式的Web后端开发

数据库

MySQL的安装与基本配置

SQL

</center>

<!--more-->

-----

# 数据库(DataBase)

## 什么是数据库？

用于存储和管理数据的仓库

## 数据库的特点

1. 持久化存储数据：其实数据库就是一个文件系统
2. 方便的管理数据：
3. 使用了统一的方式来操作数据库

## 常见的数据库软件

- MySQL：开源免费的数据库，Oracle收购
- Oracle：收费的大型数据库，Oracle公司的产品
- DB2：IBM公司的数据库产品，收费的，常用在银行系统
- SQLServer：微软公司的中型数据库
- DB2：IBM公司

# MySQL

## 安装
[安装指南](https://zhuanlan.zhihu.com/p/46905335)

但我依然配置了图形界面，并且在环境变量里的系统变量配置了`MySQL`的实际`bin`路径

## 卸载

假如没有安装成功，那么需要先卸载再重新安装

1.  卸载MySQL
2.  删除`C:\ProgramData\MySQL`此文件


## 配置

* MySQL服务启动
    + cmd中输入`Services.msc`，打开图形界面启动关闭MySQL服务
    + 以管理员运行cmd，输入`net stop mysql80`来停止，`net start mysql80`来运行

## 登录与退出

### 本地MySQL的登录
* 登录
```
mysql -uroot -p
```
-u后是用户名，用户名都是root

-p后是密码，可以此处直接输入也可以下一步再输入

* 退出
```
exit
//quit也可以
```

### 连接他人的MySQL
```
mysql -h127.0.0.1 -uroot -p
//mysql --host=127.0.0.1 --user=root --password也可以
```
-h后面填IP地址

-p后是目标的密码



## MySQL的目录结构

### 安装目录
* `bin`：二进制可执行文件
* `include`：c语言头信息
* `lib`：所需库文件
* `share`：错误信息

### 数据目录
* 数据库：文件夹
* 表：文件
* 数据：文件的内容



# SQL

## 什么是SQL？

Structured Query Language：结构化查询语言

定义了操作所有**关系型数据库**的通用规则

每一种数据库操作的方式存在不一样的地方，称为方言

## SQL通用语法

1. SQL语句以分号结尾，可以单行或者多行书写
2. MySQL数据库的SQL语句不区分大小写，但是建议使用大写
3. 注释
    1. 单行注释：`-- 注释内容`MySQL独有`#注释内容`
    2. 多行注释1：/*注释*/
```
show databases; -- 你好
//注意空格
show databases; #你好
//MySQL独有方法不需要空格
```

## 分类
按功能分为四类
1. DDL(Data Definition Language)
   
    数据定义语言，用来定义数据库对象：数据库、表、列等。

    关键字`create` `drop` `alter`
2. DML(Data Manipulation Language)

    数据操作语言，用来对数据库中表的数据进行增删改
    
    关键字`insert` `delete` `update`
3. DQL(Data Query Language)

    数据查询语言，用来查询数据库表的数据
    
    关键字`select` `where`
4. DCL(Data Control Language)(了解即可)

    数据控制语言，用来定义数据库的访问权限和安全级别及创建用户
    
    关键字`GRANT` `REVOKE`
## DDL：操作数据库、表

### 操作数据库的CRUD
* Create增
    * `create database 库名称;`增加一个数据库
    * `create database if not exists 库名称;`先判断是否存在改库再创建库
    * `create database 库名称 character set utf8;`创建时指定字符集，MySQL8.0之后默认字符集是utf8mb4，是utf8的一个超集

* Delete删
    * `drop database 库名称;`删除一个数据库

* Update改
    * `alter database 库名称 character set utf8;`更改数据库的字符集

* Retrieve查
    * `show databases;`查询所有的数据库
    * `show create database 库名称;`查询某一个数据库的创建语句，可以看到字符集

* 使用数据库
    * `select database();`查询当前正在使用的数据库名称
    * `use 库名称;`使用某库

### 操作表的CRUD



* Create增
    * `create table 表名(列名1 数据类型1,列名2 数据类型2...列名n 数据类型n);`创建一个表，注意最后一列不要加逗号
    * 常用数据类型：
        * int：整数类型
        * double：小数类型
            * `列表名 double(小数有几位,小数点后保留几位) `
        * date：日期类型，yyyy-MM-dd
        * datetime:日期类型：yyyy-MM-dd HH:mm:ss
        * timestamp:时间戳类型，yyyy-MM-dd
            * 如果不给这个字段赋值或赋值为null，则默认使用当前系统时间来默认赋值
        * varchar：字符串类型
            * `列名 varchar(20)`最大二十个字符
    * `create table 新表 like 旧表`复制表


* Delete删
    * `drop table 表名称;`删除表名称
    * `drop table if exists 表名;`判断删除

* Updata改
    * `alter table 表名 rename to 新的表名`修改表名
    * `show create table 表名;`查看表的字符集
    * `alter table 表名 character set 字符集名称;`修改表的字符集
    * `alter table 表名 add 列名 数据类型;`添加列
    * `alter table 表名 change 旧表名 新表名 数据类型`修改列名及类型
    * `alter table 表名 modify 列名 类型;`只改类型
    * `alter table 表名 drop 列名`删除列

* Retrieve查
    * `show tables;`查询当前库内所有的表
    * `desc 表名称`查询表结构

### Navicat
使用命令行来处理数据是非常麻烦的，

所以以后我们使用Navicat来对数据进行操作

## DML：增删改表中的数据

* 添加数据：
    * `insert into 表名(列名1，列名2，...，列名n) values(值1，值2，...，值n);`
        * 注意要一一对应
        * 不定义列名会给所有值添加此值
        * 除数字外都得加引号

* 删除数据：
    * `delete from 表名 where 条件`
        * 不加条件会删除所有
        * 如果要删除所有记录不要用此方法
    * `truncate table stu;`删除表再创建一个一模一样的空表

* 修改数据
    * `update 表名 set 列名1=值1,列名2=值2,... where 条件`
        * 不加条件会修改所有

