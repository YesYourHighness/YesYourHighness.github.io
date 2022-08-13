---
title: Tomcat
date: 2019-09-03 17:12:06
tags: 
- Tomcat
categories: 
- 后台
- Tomcat
---

<center>
引言：

服务器软件Tomcat

</center>


<!--more-->
----

# 服务器

服务器：安装了服务器软件的计算机

服务器软件：接收客户的请求，处理请求，做出响应

## web服务器软件

- 在web服务器软件中，可以部署web项目，让用户通过浏览器访问这些项目

- web的服务器软件也叫web容器
- 常见JavaWeb服务器软件
    1. **webLogic** :Oracle公司，大型的JavaEE服务器，收费，支持所有的JavaEE规范
    2. **webSphere**：IBM公司，大型的JavaEE服务器，收费
    3. **JBOSS**：JBOSS公司，同上
    4. **Tomcat**：Apache基金组织，中小型的JavaEE服务器，仅仅支持少量的JavaEE规范，开源，免费

（JavaEE：Java语言在企业级开发中使用的技术规范的总和，一共规定了13项大的规范）


# Tomcat

## Tomcat目录结构
- bin：可执行文件
- conf：配置文件
- lib：依赖jar包
- logs：日志文件
- temp：临时文件
- webapps：存放web项目
- work：存放运行时数据

## 操作

### 启动：

bin目录下的`starup.bat`，双击运行该文件

浏览器输入`http://localhost:8080`访问

### 可能遇见的问题
- 黑窗口闪退
    
    原因：没有正确配置JAVA_HOME环境变量
    
    解决：配置JAVA_HOME
- 打开报错
    
    原因：端口号被占用

    解决：
    1. 找到占用的端口号杀死该进程
        
        `cmd`中输入`netstat -ano`找到占用端口号的程序，记下它的PID，打开任务管理器找到对应PID结束进程
    2. 修改Tomcat自身的端口号
        
        conf文件夹下的`server.xml`文件，找见`connector`标签，更改它的port，一般改为80

### 关闭

1. 正常关闭：双击`shutdown.bat`或者输入`CTRL+C`

2. 强制关闭：点击启动窗口的关闭按钮

### 部署
- 部署项目的方式

1. 直接将项目放到webapps目录下即可
        
    `/xxx`项目的访问路径-->也叫虚拟目录
            
    简化配置：将html文件压缩为war格式，放在webapps内，会自动的装载出该文件的目录，删除war包也会自动删除对应的文件目录

2. 更改`server.xml`配置文件

    ```xml
    <Context docBase="项目路径" path="虚拟目录访问路径" />
    ```
    这样就部署完成了

    但这样配置很不安全，容易把配置文件搞坏
    
3. 在`conf\Catalina\localhost`目录下创建xml文件

    输入
    ```xml
    <Context doBase="项目目录" />
    ```
    **虚拟目录是xml文件的名称**

## 静态项目和动态项目

目录结构：

### Java动态项目的目录结构

- 项目的根目录
    - WEB-INF目录
        - `web.xml`：web项目的核心配置文件
        - `classes`目录：放置字节码文件的目录
        - `lib`目录：放置依赖的jar包

## 将Tomcat集成到IDEA中
将Tomcat集成到IDEA中并且创建JavaEE的项目，部署项目

- 将Tomcat集成到IDEA

1. 打开最上边的`run`,再进入`Edit configurations`
2. 在`Templates`找见`Tomcat server`
3. 配置`Application server`为Tomcat的目录
4. 点击ok
5. 创建项目
6. 勾选Java Enterprise
7. 勾选java版本7，为了学习之后的内容
8. 勾选`web application`
9. 点击创建
10. 再回到第二步，勾选两个都为`Update resources`热更新
11. 启动项目即可