---
title: Spring_IOC入门
date: 2020-03-07 23:16:49
tags:
- Spring
categories:
- Spring
---

<center>
引言：

Spring_框架入门
</center>

<!--more-->


# Spring是什么

> Spring是分层的 Java SE/EE应用 full-stack **轻量级**开源框架, 以 IoC（Inverse Of Control： 反转控制）和 AOP（Aspect Oriented Programming：面向切面编程）为**内核**，提供了展现层 Spring MVC 和持久层 Spring JDBC 以及业务层事务管理等众多的企业级应用技术，
还能整合开源世界众多著名的第三方框架和类库，逐渐成为使用最多的Java EE 企业应用开源框架


- 轻量级：与EJB对比，依赖资源少
- 分层：经典三层
    - web层：struts，spring-MVC
    - service层：spring
    - dao层：hibernate，mybatis，jdbcTemplate
- 核心：IOC和AOP

看不懂一点也没关系，不影响看底下的内容


# Spring各模块一览
浏览一下就好，都不认识没关系，有认识的那就更开心了

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/Spring/springModel.png)



# IOC

先来了解一下IOC的概念
> Inverse Of Control 字面意思：控制反转

那么就有两个问题：
- IOC控制什么？
    **控制对象的创建及销毁**（生命周期）
- IOC反转什么？
    **将对象的控制权交给IOC容器**


## 为什么要用IOC
为什么要使用IOC，要从开发的演变来说起。


假设这样一个情形：张三要开一辆奥迪车回家

根据**面向对象的思想(OOP)**，我们需要创建两个类：张三类和奥迪类

- 奥迪车的类
```java
public class Audi {
    public void start(){
        System.out.println("车辆启动");
    }
    public void stop(){
        System.out.println("车辆停止");
    }
    public void turnLeft(){
        System.out.println("车辆左转向");
    }
    public void turnRight(){
        System.out.println("车辆右转向");
    }
}
```
- 张三的类
```java
public class ZhangSan {
    public void goHome(){
        Audi audi = new Audi();
        audi.start();
        audi.turnLeft();
        audi.turnRight();
        audi.stop();
    }
}
```
张三是个程序员，最近996熬夜肝项目，挣了好多钱，又买了辆别克

- 新建一个别克车的类
```java
public class Buick {
    public void start(){
        System.out.println("车辆启动");
    }
    public void stop(){
        System.out.println("车辆停止");
    }
    public void turnLeft(){
        System.out.println("车辆左转向");
    }
    public void turnRight(){
        System.out.println("车辆右转向");
    }
}
```
但是张三的类我们还没改，如果要改的话，我们得把所有出现`audi`的地方改成`buick`，这显然是麻烦的，所以我们提出一个首行域
```java
public class ZhangSan {
    Audi audi = new Audi();
    public void goHome(){
        audi.start();
        audi.turnLeft();
        audi.turnRight();
        audi.stop();
    }
}
```
这样的话我们就可以轻松的改掉第一行为`Buick audi = new Buick();`，其他的保留原样，就实现了

但我们转念一想：

张三是程序员，只是需要一辆车，不同类型的车对他来说没什么不同

所以我们要进行改动

- 新增一个`Car`接口，让`Audi`和`Buick`都实现`Car`这个接口
```java
public interface Car {
    public void start();

    public void stop();

    public void turnLeft();

    public void turnRight();
}
```
- 张三的类，让`Car`从构造方法中传入
```java
public class ZhangSan {
    protected Car audi;

    public ZhangSan(Car audi) {
        this.audi = audi;
    }

    public void goHome(){
        audi.start();
        audi.turnLeft();
        audi.turnRight();
        audi.stop();
    }
}
```
这样我们就解决了刚才的两个问题，这就是**面向接口的思想**

但此时，我们发现，有些不对劲，车还是由张三造的，作为一个996的程序员，他哪里会造什么车子呢

所以我们需要新建一个`CarFactory`这个工厂类
```java
public class CarFactory{
    public static Car getCar(Car car){
        return new Car car;
    }
}
```
这就是**工厂模式**，这样我们张三就和造车没有关系了，可以安安心心的敲代码了

可是我们的张三和车厂有了关系（耦合），下次我们

有了新的要求，我们还得去改代码，这样不符合**OCP原则(Open-Close Protocol)**，我们要求对程序扩展是open的，对修改程序代码时close的，尽量做到不修改程序的原码，而是修改对程序的扩展


那么这时候，车辆该由谁来创建呢？ 该由`IoC`容器来创建，我们大大方方的交给`Spring`，让它来帮我们处理这个问题

`Spring`其实就是使用**工厂+反射+配置文件**

类似这样
```xml
<bean id="car" class="com.xxx.xxx.Car"></bean>
```
```java

class CarFactory{
    public static Obejct getBean(String id)
    ...
    反射
}
```


## 再来认识IOC

> Inverse Of Control 字面意思：控制反转

他有两个特点
- IOC控制：
    **控制对象的创建及销毁**（生命周期）
- IOC反转：
    **将对象的控制权交给IOC容器**

在官方文档中，有这么一段话
>  IoC is also known as dependency injection (DI).

`IOC`也被称为`DI`依赖注入，DI的意思是，对象的属性也可以在配置文件中定义，然后在`Spring`创建这个对象的同时，将这个对象所依赖的属性注入


# 参考资料
> 慕课网 及 官方文档