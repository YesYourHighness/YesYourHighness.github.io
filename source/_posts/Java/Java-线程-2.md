---
title: Java-线程-2
date: 2019-08-18 15:44:38
tags: 
- Java
- Java线程
categories: 
- 后台
- Java
---
<center>
引言：

创建线程的另一种方法：

Runnable接口

线程安全
</center>

<!--more-->

--------

# Runnable

创建线程的另一种方法：使用Runnable接口

开启线程的两个构造
```java
Thread(Runnable target)
//分配新的Thread对象

Thread(Runnable target,String name)
//分配新的Thread对象
```

示例代码:

实现类
```java
//创建Runnable接口的实现类
public class runnable implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println("另一种方法"+i);
        }
    }
}
```
main函数
```java
public class demo {
    public static void main(String[] args) {
        runnable r = new runnable();
        //实现类实例化对象
        new Thread(r).start();
        //通过Thread类对象来开启新线程
        for (int i = 0; i < 3; i++) {
            System.out.println("main"+i);
        }
    }
}
```

# 两种方法的区别

区别：

实现Runnable接口创建多线程的好处：
1. 避免了单继承的局限性（ 以接很多个）
2. 增强了程序的扩展性，降低了程序的耦合性（解耦）
    + 实现Runnable接口的方式，把设置线程任务和开启新线程进行了分离
        - 实现类重写run方法实现设置线程任务
        - 创建Thread对象，调用start方法，来开启新的线程

# 多线程安全

在多线程访问共享数据时，容易发生错误

解决安全问题：同步技术——有三个

## 同步代码块

锁对象：任意的对象，但是必须保证多个线程使用的锁对象是同一个

作用：把同步代码块锁住，只让一个线程在同步代码块中执行

```java
synchronized ("同步锁"){
    //需要同步的代码
}
```
示例
```java
public class runnable implements Runnable {
    private int ticket = 10;
    Object obj = new Object();
    @Override
    public void run() {
        while (true) {
            synchronized (obj){
                if (ticket > 0) {
                    System.out.println(Thread.currentThread().getName()+"   "+ticket);
                    ticket--;
                }
                else return;
            }
        }
    }
}
```


## 同步方法

- 把访问了共享数据的代码放到方法中
- 加synchronized关键字
- 锁对象是this
```java
public synchronized void method(){
    
}
```
示例
```java
public class runnable implements Runnable {
    private int ticket = 10;
    Object obj = new Object();
    @Override
    public void run() {
        while (true) {
            method();
        }
    }
    public synchronized void method(){
        if (ticket > 0) {
            System.out.println(Thread.currentThread().getName()+"   "+ticket);
            ticket--;
        }
        else return;
    }
}
```
静态同步方法

- 静态没有this
- 锁对象：本类的class属性—class文件对象（反射）
```java
public class runnable implements Runnable {
    private static int ticket = 10;
    Object obj = new Object();
    @Override
    public void run() {
        while (true) {
            method();
        }
    }
    public static synchronized void method(){
        if (ticket > 0) {
            System.out.println(Thread.currentThread().getName()+"   "+ticket);
            ticket--;
        }
        else return;
    }
}
```

## Lock锁接口
包路径
```
java.util
```
方法：
```java
lock()//获取锁
unlock()//解锁
```
步骤：
1. 在成员位置创建一个Reentrantlock对象
2. 在可能会出现安全问题的代码调用lock()方法
3. 在可能会出现安全代码问题之后调用unclock()方法
```java
public class runnable implements Runnable {
    private int ticket = 10;
    Lock l = new ReentrantLock();
    @Override
    public void run() {
        while (true) {
            l.lock();
            if (ticket > 0) {
                System.out.println(Thread.currentThread().getName()+"   "+ticket);
                ticket--;
            }
            l.unlock();
        }
    }
}
```
## 同步技术的原理：

使用了一个**锁对象**，这个锁对象叫同步锁，也叫对象锁，也叫对象监视器

当发生多线程分享共有资源时，抢到cpu运行权的时候，

运行到synchronize时，检查是否有锁对象，
有，
就会获取到锁对象并进入到同步过程中，运行完归还锁对象。

没有，就会等待直到锁对象被归还。