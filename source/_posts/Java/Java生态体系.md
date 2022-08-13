---
title: Java生态体系
date: 2019-08-11 17:23:00
tags: 
- Java
categories: 
- 后台
- Java
---
<center>
引言：

安装java环境

认识java整个生态体系

</center>

<!--more-->

----

# Java
Java的名字是一个咖啡的名字，没错，牛人就是这么随意，同样的Python也是起源于一部美剧。

Sun公司创造了Java，后来Oracle公司收购了Java，我们现在所使用的基本都是Oracle公司的java

## Java环境变量的配置

去Oracle官网寻找java1.8版本下载

下载完成后安装

1. 安装完java后要配置环境变量
2. 在电脑属性中打开高级系统设置
3. 再打开环境变量配置

在环境变量中的系统变量内输入
```
JAVA_HOME       C:\xxxxxxjava安装路径
```

再修改一下PATH的内容
```
%JAVA_HOME%\bin;
```
添加上述即可，注意不要忘记分号

## Java技术体系

1. Java设计语言
2. 各种平台上的Java虚拟机（JVM）
3. Java API类库
4. 一系列的辅助工具，如javac
5. `1+2+3+4`=JDK(Java开发最小环境)
6. `2+3`=JRE(Java运行最小环境)
7. JDK > JRE > JVM

## Java技术体系所划分的三大平台
1. JavaSE：标准版的SE，用来开发一些电脑上的桌面软件等等
2. JavaEE：企业版的Java，也是我们要一直学习的
3. JavaME：微型Java，用来进行嵌入式开发


## JVM/JRE/JDK


JVM，JRE，JDK傻傻分不清楚，他们到底都是什么东西

### JVM
全名：Java Virtual Machine

翻译就是**java虚拟机**的意思

运行所有JAVA程序的虚拟计算机，不同的电脑装入不同的虚拟机，

而JAVA程序就可以实现运行在JVM上面，从而实现跨平台的使用这个程序，

但是要注意，虚拟机本身不是跨平台

### JRE
全名：Java Runtime Environment

Java运行时环境

Java运行时所处于的环境，包含JVM和运行时所需要的核心类库

### JDK
全名：Java Development Kit

Java开发工具包

开发Java程序所必要的环境

## JavaEE

学习JavaEE我们都需要学习什么呢

### 框架

#### Spring：
 一个开源的应用程序框架，提供了一个简易的开发方式，说的更通俗一点就是由框架来帮你管理这些对象，包括它的创建，销毁

 Spring就像是整个项目中装配bean（javabean是指java中可重复利用的组件）的大工厂，

 在配置文件中可以指定使用特定的参数去调用实体类的构造方法来实例化对象。
#### SSH
~~Spring+Struct+Hibernate~~(过时，不学)
#### SSM
SSM指：Spring+SpringMVC+Mybatis

- **SpringMVC**
  
    它原生支持的Spring特性，让开发变得非常简单规范。
    
    Spring MVC 分离了控制器、模型对象、分派器以及处理程序对象的角色，这种分离让它们更容易进行定制。


- **Mybatis**

    可以这么理解，MyBatis是一个用来帮你管理数据增删改查的框架。。

#### SpringBoot
实现自动配置，降低项目搭建的复杂度

初期的Spring通过代码加配置的形式为项目提供了良好的灵活性和扩展性，

但随着Spring越来越庞大，其配置文件也越来越繁琐，太多复杂的xml文件也一直是Spring被人诟病的地方，

大家渐渐觉得Spring那一套太过繁琐，

此时，Spring社区推出了Spring Boot，它的目的在于实现自动配置，降低项目搭建的复杂度

#### SpringCloud
一系列框架的有序集合。

它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发

### 各种中间件

- MQ消息队列
- RPC通信框架
- 搜索引擎elasticsearch

### 数据库

- SQL：MySQL/ Postgre SQL
- NoSQL：Redis Memcacher mongodb elasticserach


### 架构
- 分布式/微服务架构
- spring cloud
- dubbo
- rpc通信框架

### 虚拟化/容器化技术
- Docker 容器化
- K8S 
---
### 源码/性能（难度最高）
- JDK源码以及部分设计思想
- Spring源码
- JVM细节与排错
- 高并发/高可用

