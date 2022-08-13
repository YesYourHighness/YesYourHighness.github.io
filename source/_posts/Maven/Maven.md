---
title: Maven
date: 2020-01-12 11:47:14
tags: 
- Maven
categories:
- Maven
---

<center>
引言： 

Maven：项目管理工具

</center>

<!-- more -->


# Maven

## Maven

Maven是什么？
> Maven 是一个项目管理工具，可以对 Java 项目进行构建、依赖管理。

为什么要学习Maven？
1. 帮我们处理jar包之间的冲突
2. 帮我们进行编译
3. 帮我们进行单元测试
4. 整合项目


> 依赖管理：Maven工程将开发所用的jar包放置在jar包仓库中，通过在 pom.xml 文件中添加所需 jar 包的坐标，而传统模式下会把所有的jar包都放在项目里

> 一键构建：Maven只需要一行命令就可以实现从编译、测试、运行、打包、安装、部署的一系列过程

## 安装

1. 去Maven的[官网](https://maven.apache.org/download.cgi?Preferred=https%3A%2F%2Fwww-eu.apache.org%2Fdist%2F)，进行下载

2. 下载完成后，解压到某个位置

3. 配置系统变量和Path路径

系统变量
![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic1/maven_setting01.png)
Path路径
![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic1/maven_setting03.png)

4. 配置完成后打开cmd，输入`mvn -v`，显示版本信息，说明安装成功

## Maven仓库

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic1/maven_repository.png)

如图，Maven三个仓库
- 本地仓库：无需网络，访问本地的jar包仓库，默认位置在` ${user.dir}/.m2/repository`但我们都会自己配置这个地址
- 远程仓库：局域网连接，公司内部使用的一个仓库
- 中央仓库：需要互联网，这里几乎有所有的jar包，远程仓库地址 https://mvnrepository.com/ 

## 配置

### 全局setting与用户setting
- 全局配置：在 maven 安装目录下的有 `conf/setting.xml` 文件，此 `setting.xml` 文件用于 maven 的所有 project 项目，它作为 maven 的全局配置。 

- 局部配置：如需要个性配置则需要在用户配置中设置，用户配置的 `setting.xml` 文件默认的位置在：`${user.dir} /.m2/settings.xml` 目录中,${user.dir} 指windows 中的用户目录。

**Maven会先找用户配置，如果找到则以用户配置为准，否则使用全局配置**

### 我们要进行的配置

我们需要将需要的jar包保存到一个本地仓库，然后更改如下配置：

1. 打开conf目录下的`settings.xml`文件，找到如图位置

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic1/maven_changeRepository_position.png)

2. 将这一段复制出来，粘贴到底下

```html
<localRepository>/path/to/local/repo</localRepository>
```
3. 标签中间内容输入你的本地Maven仓库的地址，如

```html
<localRepository>D:\maven\maven_repository</localRepository>
```
到此Maven的配置就完成了

## Maven标准目录结构

一个开发完成的项目，分为几部分：
1. 核心代码部分
2. 配置文件部分——这一部分可能会需要频繁的修改，所以不需要打为jar包
3. 测试代码
4. 测试配置文件

### 传统的目录结构：
```
项目名
    -src
    -config
    -xxxx
```
没有统一的规定，基本上所有的文件堆放在src目录下，混乱复杂，打包容易把所有的文件都打包


### Maven的目录结构

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic1/maven_struct01.png)
![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic1/maven_struct02.png)
```
src/main/java      //放置核心代码
src/main/resources //放置配置文件
src/main/webapp    //放置页面资源js，css,img资源
//注意：如果是普通的 java 项目，那么就没有webapp 目录

src/test/java      //放置测试代码
src/test/resources //放置测试配置文件

target —— 项目输出位置，编译后的class 文件会输出到此目录 
pom.xml——maven 项目核心配置文件 
```

## Maven常用的命令
1. 编译命令
```
mvn compile
```
 maven 工程的编译命令，作用是将 `src/main/java` 下的文件编译为 `class 文件`输出到 `target` 目录下。
 
 2. 测试命令
```
mvn test
```
test 是 maven 工程的测试命令`mvn test`，会执行`src/test/java`下的单元测试类。 
3.  清除命令
```
mvn clean
```
clean 是 maven 工程的清理命令，执行 clean 会删除 `target 目录及内容`。 

4. 打包命令
```
mvn package
```
`package`是`maven`工程的打包命令，对于 java 工程执行 package 打成`jar`包，对于web 工程打成`war`包。 
5. 安装命令
```
mvn install
```
执行 install 将 maven 打成 jar 包或 war 包发布到本地仓库。 


**当后面的命令执行时，前面的操作过程也都会自动执行**

6. 运行Maven工程

进入 maven 工程目录（当前目录有 pom.xml 文件），运行
```
tomcat:run
```

## Maven的生命周期

maven 对项目构建过程分为**三套相互独立**的生命周期，请注意这里说的是“三套”，而且“相互独立”， 这三套生命周期分别是：  
 
- Clean Lifecycle 在进行真正的构建之前进行一些清理工作。  
- Default Lifecycle 构建的核心部 分，编译，测试，打包，部署等等。
- Site Lifecycle 生成项目报告，站点，发布站点

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic1/maven_lifeCycle.png)

## Maven 概念模型

Maven包含了：
- 项目对象模型 (Project Object Model)
- 一组标准集合
- 一个项目生命周期(Project Lifecycle)
- 一个依赖管理系统(Dependency Management System)
- 和用来运行定义在生命周期阶段 (phase)中插件(plugin)目标(goal)的逻辑

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic1/maven_gainian.png)

### POM
>项目对象模型(Project Object Model)

每个Maven工程都有一个`pom.xml`文件，配置了项目的坐标、项目依赖、插件信息等 例如：JDK，Tomcat等


### DMS
> 依赖管理系统(Dependency Management System)

通过 maven 的依赖管理对项目所依赖的 jar 包进行统一管理。 

例如：项目依赖 junit4.9，通过在 pom.xml 中定义 junit4.9 的依赖即使用 junit4.9，如下所示是 junit4.9 的依赖定义： 
```html
<!-- 依赖关系 -->  
<dependencies>
    <!-- 此项目运行使用 junit，所以此项目依赖 junit -->   
    <dependency>    
        <!-- junit 的项目名称 -->    
        <groupId>junit</groupId>    
        <!-- junit 的模块名称 -->    
        <artifactId>junit</artifactId>    
        <!-- junit 版本 -->    
        <version>4.9</version>    
        <!-- 依赖范围：单元测试时使用 junit -->
        <scope>test</scope>   
    </dependency> 
</dependencies>
```
 
### 项目生命周期(Project Lifecycle)  
 使用 maven 完成项目的构建，项目构建包括：清理、编译、测试、部署等过程，maven 将这些 过程规范为一个生命周期

### 一组标准集合  
maven将整个项目管理过程定义一组标准
比如：通过 maven 构建工程有标准的目录结构，有 标准的生命周期阶段、依赖管理有标准的坐标定义等。 
 
### 插件(plugin)目标(goal) 
maven 管理项目生命周期过程都是基于插件完成的
