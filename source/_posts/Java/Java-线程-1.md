---
title: Java-线程-1
date: 2019-08-18 15:44:28
tags: 
- Java
- Java线程
categories: 
- 后台
- Java
---

<center>
引言：

多线程是什么？

Thread类
</center>

<!--more-->

---

# 多线程

## 并发和并行
- 并发：指两个或多个事件在同一个时间段内发生

- 并行：指两个或多个时间在同一时刻发生（同时发生，更快一些）

## 进程

一个内存中运行的应用程序，每个进程都有一个独立的内存空间，一个应用程序可以同时运行多个进程；

进程也是程序的一次执行过程，是系统运行的基本单位

系统运行一个程序就是一个进程从创建到消亡的过程


## 线程
程序到cpu的一个执行路径

进程的一个执行单元，

负责当前进程中程序的执行，

一个进程中至少有一个线程，

一个进程中是可以有多个线程的，这个应用程序也可以称之为多线程程序。

### 线程调度
- 分时调度

所有线程轮流使用cpu的使用权，平均分配每个线程占用cpu的时间
- 抢占式调度

优先让优先级高的线程使用cpu，如果优先级相同，那么会随机选择一个（线程随机性），java使用的为抢占式调度。


## 多线程原理
1. JVM虚拟机执行`main`方法的时候，会开辟一条通往`cpu`的路径。（这个路径叫**main线程**，主线程）
2. cpu可以通过这个路径执行`main`方法。
3. 当创建`Thread`类的子类对象的时候，会开辟另一台通往cpu的路径。
4. 两条路径，cpu进行选择，我们无法控制

每调用一次Thread类的子类的start方法都会开辟一个新的栈空间用来执行run方法

- 多线程的好处：

    多个线程之间互不影响（在不同的栈空间）

# Thread

线程是程序中的执行线程。

Java虚拟机允许应用程序并发地运行多个执行线程。 

每个线程都有一个优先级，高优先级线程的执行优先于低优先级线程。

## 包路径
```java
java.lang
```

## 常用方法

- 获取线程的名称：
1. 使用Thread类中的方法getName()
```java
public final String getName()
//返回该线程的名称。 
```
2. 先获取到当前正在执行的线程，使用线程中的方法`getName()`获取线程的名称
```java
 public static Thread currentThread()
 //返回对当前正在执行的线程对象的引用。 
```
第一种获取线程名字方法：
```java
public class thread extends Thread {
    @Override
    //重写Thread类中的run方法，设置线程任务
    public void run() {
        System.out.println(getName());
    }
}
```
主函数调用`start()`方法
```java
public class demo {
    public static void main(String[] args) {
        thread my = new thread();
        my.start();//Thread-0
        new thread().start();//Thread-1
        new thread().start();//Thread-2
        new thread().start();//Thread-3
    }
}
```
第二种获取线程名字的方法：
```java
public class thread extends Thread {
    @Override
    public void run(){
        Thread t = Thread.currentThread();
        System.out.println(t);//Thread[Thread-0,5,main]
        System.out.println(t.getName());//Thread-0
    }
}
```
```java
public class demo {
    public static void main(String[] args) {
        thread my = new thread();
        my.start();
    }
}
```

- 设置线程的名字
1. 使用Thread的setName()名称
2. 创建一个带参数的**构造方法**，参数传递线程名称，调用父类的带参构造方法，让父类给子线程起一个名字

法一：
```java
thread my = new thread();
my.setName("线程1");
my.start();
```
法二：
```java
public class thread extends Thread {
    public thread(){}

    public thread(String name){
        super(name);
    }

    @Override
    public void run(){
        System.out.println(getName());
    }
}
```
```java
thread my = new thread("线程1");
my.start();
```

- 指定秒数停止程序
```java
public static void main(String[] args) {
    for (int i = 0; i <= 60 ; i++) {
        System.out.println(i);
        try {
            Thread.sleep(1000);//休眠1秒
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```