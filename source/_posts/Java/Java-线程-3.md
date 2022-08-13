---
title: Java-线程-3
date: 2019-08-18 15:44:41
tags: 
- Java
- Java线程
categories: 
- 后台
- Java
---
<center>
引言：

线程的状态

线程池
</center>

<!--more-->

---


# 线程的状态

## 状态
状态| 意义
---|---
new |新建状态 
Runnable | 运行状态	
Blocked | 阻塞状态
Terminated | 死亡状态
Timed_waiting|休眠状态
Waiting | 无线等待状态

## 线程图

![image](https://img-blog.csdn.net/20180502203945924?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwNjA0OTg5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


## wait/notify
Object的两个方法：

进入到TimeWating(计时等待)有两种方式：
1. 使用`sleep(long m)`方法，在毫秒值结束之后，线程睡醒进入到`Runnable/Blocked`状态
2. 使用`wait(long m )`方法，`wait`方法如果在毫秒值结束之后，还没被`notify`唤醒，就会自动醒来
```java
public class demo {
    public static void main(String[] args) {
        //创建一个锁对象
        Object obj = new Object();
        //创建一个顾客线程
        new Thread() {
            @Override
            public void run() {
                //保证只有一个执行
                synchronized (obj) {
                    System.out.println("顾客：我要一个包子");
                    try {
                        obj.wait(5000);
                        //Thread.sleep(5000);同样可以
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("老板：包子做好了");
                    System.out.println("顾客：吃包子");
                }
            }
        }.start();
        new Thread() {
            @Override
            public void run() {
                System.out.println("老板：做包子5s");
                synchronized (obj) {
                }
            }
        }.start();
    }
}
```
唤醒的方法也有两个：
1. `notify()`方法：随机唤醒一个
2. `notifyAll()`方法：唤醒所有的等待

注意事项：
1. `wait`方法与`notify`方法必须同一个锁对象调用！！
2. `wait`和`notify`方法是属于`Object`类的方法的
3. `wait`和`notify`必须要在同步代码块或者是同步函数中使用

# 线程池
我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但就会有一个问题：

如果并发的线程数很多，并且每一个线程都是执行一个时间很短的任务就结束了，

这样频繁的创建线程就会大大降低系统的效率，因为频繁的创建和销毁线程需要时间

线程池就可以用来执行完一个任务而不被立刻销毁，而是可以继续完成其他任务

**线程池**：

容纳多个线程的容器——集合:

JDK1.5之后,JDK内置了线程池，我们可以直接使用

好处：
1. 减少了资源消耗，减少了线程创建和销毁的次数
2. 提高响应速度：任务不需要等待线程创建即可执行
3. 提高可管理性：根据系统的承受能力，调整线程池中工作线程的数目（每个线程大约要用1MB的内存）

## Executors
### 包路径
```java
java.util
```
### 构造
```java
public static ExecutorService newFixedThreadPool(int nThreads)
//创建一个可重用固定线程数的线程池，
//以共享的无界队列方式来运行这些线程

//返回值是一个ExecutorService接口，返回的是ExecutorService接口的实现类对象
```
ExecutorService
线程池的接口，用来从线程池中获取线程，调用start方法执行线程任务

### ExecutorService常用方法
```java
Future<?> submit(Runnable task)
//提交一个 Runnable 任务用于执行，
//并返回一个表示该任务的 Future。
//该 Future 的 get 方法在成功 完成时将会返回 null

void shutdown()
//启动一次顺序关闭，执行以前提交的任务，但不接受新任务。如果已经关闭，则调用没有其他作用。
```
### 步骤

步骤：
1. 使用工厂类Executors的静态方法newFixedThread生产一个指定线程数量的线程池
2. 创建一个类，实现Runnable接口，重写run方法，设置线程任务
3. 调用ExecutorService方法submit，传递线程任务（实现类），开启线程，执行run方法
4. 调用ExecutorService方法shutdown销毁线程池（不建议执行）

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class demo {
    public static void main(String[] args) {
        //1.使用工厂类Executors的静态方法newFixedThread生产一个指定线程数量的线程池
        ExecutorService es = Executors.newFixedThreadPool(2);
        //3.调用ExecutorService方法submit，传递线程任务（实现类），开启线程，执行run方法
        es.submit(new demoImpl());
        //线程池会一直开启，使用完了线程，会自动把线程归还给线程池，线程可以继续使用
        es.submit(new demoImpl());
        es.submit(new demoImpl());
        //4.调用ExecutorService方法shutdown销毁线程池（不建议执行）
        es.shutdown();
    }
}
```
```java
//2.创建一个类，实现Runnable接口，重写run方法，设置线程任务
public class demoImpl implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"创建了一个新的线程执行");
    }
}
```

