---
title: Java 线程与并发
date: 2021-08-10 09:06:20
tags:
- JUC
- 锁
categories:
- JUC
---
<center>
  引言:
  JUC核心部分
</center>
<!-- more -->

#  Java 线程与并发

本章分为两个部分：

上半部分：Java多线程——基础

下半部分：Java并发编程——进阶

# Java多线程

## 创建线程的方式

在Java中，有**四种**创建线程的方法：

- 继承`Thread`类
  1. 继承`Thread`类
  2. 重写`run()`方法
  3. 用`start()`方法开启线程（是一个Native方法）
- 实现`Runnable`接口
  1. 实现`Runnable`接口
  2. 重写`run()`方法
  3. 使用`Thread`的构造方法，传入实现了`Runnable`接口的类对象创建对象
  4. 调用`Thread`对象的`start()`方法
- 实现`Callable`接口（一个**有返回值的线程**）
  1. 实现`Callable<T>`接口，注意有泛型
  2. 重写`call()`方法
  3. 通过`ExecutorService`对象的`submit( Callable<T> )`方法，将实现了Callable接口的`thread`上传，返回值是一个`Future`对象
  4. 通过`Future`对象的`get()`方法就可以获取到值
- 线程池（具体内容会在下一章**Java并发编程**进行介绍）
  1. 通过`Executor`来获取线程池
  2. 通过`ExecutorService`的`execute(Runnable接口)`执行任务，没有返回值
  3. 通过`ExecutorService`的`shutdown()`方法关闭线程池

第一种方式demo（过于简单可以跳过）:

```java
public class MyThread01 extends Thread{
    @Override
    public void run() {
        super.run();
        System.out.println("继承Thread实现线程");
    }

    public static void main(String[] args) {
        MyThread01 myThread = new MyThread01();
        myThread.start();
    }
}
```

第二种方式的demo（过于简单可以跳过）：

```java
public class MyThread02 implements Runnable{
    @Override
    public void run() {
        System.out.println("用接口新建线程");
    }

    public static void main(String[] args) {
        MyThread02 myThread02 = new MyThread02();
        Thread thread = new Thread(myThread02);
        thread.start();
    }
}
```

实现`Callable`接口demo：

```java
public class MyThread03 implements Callable<String> {

    @Override
    public String call() throws Exception {
        // 具有返回值的线程，重写call方法
        String[] strs = {"a","b","c","d","e"};
        return strs[new Random().nextInt(5)];
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread03 thread = new MyThread03();
        // 创建执行服务
        ExecutorService service = Executors.newFixedThreadPool(1);
        // 提交执行
        Future<String> res = service.submit(thread);
        // 使用get获取返回值
        String s = res.get();
        System.out.println(s);
        // 关闭服务
        service.shutdownNow();
    }
}
```

线程池demo：

```java
// 1 获取线程池
ExecutorService threadPool = Executors.newFixedThreadPool(10);
while(true) {
    // 2. 执行任务，这里使用了lambda表达式
    threadPool.execute(() -> {
        System.out.println(Thread.currentThread().getName() + " is running ..");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
}
```

## 静态代理模式

静态代理模式中有 **真实对象、代理对象**

- 真实对象与代理对象要**实现同一个接口**
- 代理对象要**代理真实的角色**

优点：

静态代理模式可以帮助我们处理一些其他的事情，真实对象可以专注于做本职任务

---

举例：

在多线程中，实现`Runnable`接口的类就使用了静态代理模式：

例如这个demo：真实对象——`MyThread02`、代理对象`Thread`，他们实现了同一个接口`Runnable`，然后通过代理类`Thread`代理真实对象`myThread02`，执行`run`方法（通过`start`执行）

```java
public class MyThread02 implements Runnable{
    @Override
    public void run() {
        System.out.println("用接口新建线程");
    }

    public static void main(String[] args) {
        MyThread02 myThread02 = new MyThread02();
        Thread thread = new Thread(myThread02);
        thread.start();
    }
}
```

## 线程的五大状态

老生常谈的问题，说再多不如图：

（其实这里的五大状态，应该算OS层面的线程的五大状态，具体JVM里线程的状态，后面会说）

![线程五大状态](http://img.yesmylord.cn//image-20210727000042302.png)

除此外，还要说明几点：

1. 创建状态：此时Jvm会为其分配内存空间，初始化成员变量的值
2. 就绪状态：JVM为其创建方法栈和PC
3. 运行状态：获得了CPU
4. 阻塞状态分三种情况
   - 等待阻塞：线程调用了`wait()`方法，进入等待队列
   - 同步阻塞：要获取的同步锁被别的线程占用，JVM会将这个队列放入锁池（Lock Pool）中
   - 其他阻塞：由于`sleep()`、`join()`，或者是IO请求时产生中断
5. 导致线程死亡的情况（下一节详细介绍）
   - 正常结束：`run`或`call`方法运行结束
   - 异常结束：抛出未捕获的Error或是Exception
   - 调用stop：`stop()`不建议使用，因为很容易导致死锁；官方也声明这是一个即将过时的方法。

## 终止线程的方式

终止线程有很多方式，这里主要介绍四种：

### 正常退出

程序`run()`或`call()`方法运行结束，线程正常退出

### 使用flag退出线程

大多数情况下，线程是**伺服线程**，所以我们一般**使用一个变量**来控制线程的退出：

> 伺服线程：即需要长时间运行的线程，多为循环体

```java
public class ThreadSafe extends Thread {
    public volatile boolean exit = false;
    public void run() {
        while (!exit){
            //do something
        }
    }
}
```

注意到，此变量使用了`volatile`关键字，可以使同一时刻只能有一个线程修改`exit`的值（此关键字看下文详细阐述）

### 使用Interrupt

使用`interrupt`中断线程有两种情况：

1. 线程处于阻塞状态：

   - 当调用线程的 `interrupt()`方法时，会抛出 `InterruptException`异常。阻塞中的那个方法抛出这个异常，通过代码捕获该异常，然后 break 跳出循环状态
   - **注意**只有当**捕获异常并`break`后，才能正常结束`run`方法**

2. 线程不处于阻塞状态：使用`isInterrupted()`判断线程的中断标志来退出循环。当使用`interrupt()`方法时，中断标志就会置`true`

   ```java
   public class ThreadSafe extends Thread {
       public void run() { 
           while (!isInterrupted()){ 
               //非阻塞过程中通过判断中断标志来退出
               try{
                   Thread.sleep(5*1000);
                   //阻塞过程捕获中断异常来退出
               }catch(InterruptedException e){
                   e.printStackTrace();
                   break;//捕获到异常之后，执行 break 跳出循环
               }
           }
       } 
   }
   ```

### 使用stop

一个已经过时的方法，线程不安全 0

`thread.stop()`调用之后，创建子线程的线程就会抛出`ThreadDeatherror `的错误，并且会**释放子线程持有的隐式锁**。

一般任何进行加锁的代码块，都是为了保护数据的一致性，如果在调用 `thread.stop()`后导致了该线程所持有的所有锁的突然释放(不可控制)，那么被保护数据就有可能呈现不一致性，其他线程在使用这些被破坏的数据时，有可能导致一些很奇怪的应用程序错误。

## `sleep()`与`wait()`

- `sleep()`方法在`Thread`类中，是一个**本地静态方法**

```java
public static native void sleep(long millis) throws InterruptedException;
```

- `wait()`方法是在`Object`类中的，是一个**不可重写的本地方法**

```Java
public final native void wait(long timeout) throws InterruptedException;
```

| 对比项           | sleep                                           | wait               |
| ---------------- | ----------------------------------------------- | ------------------ |
| 是否让出CPU      | 是                                              | 是                 |
| 是否让出对象锁   | **不释放**                                      | **释放**           |
| 如何进入就绪状态 | 设定时间到或是调用`interrupt()`方法唤醒休眠线程 | 调用`notify`方法   |
| 使用范围         | 任何地方                                        | 必须在同步代码块中 |

> `wait`是醒着的等待，所以会释放锁
>
> `sleep`抱着锁睡着了，所以不会释放锁 

## `start`方法

在Java源码中，`start()`方法会调用本地方法`start0()`，由C来实现线程的创建

> 所以Java本质上来说，是创建不了线程的，需要调用C++来实现

源码如下：

```java
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();
    group.add(this);
    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
it will be passed up the call stack */
        }
    }
}

private native void start0();
```

# Java并发编程

## JUC

> 并发编程离不开JUC，什么是JUC？

指JDK下的包：`java.util.concurrent`，简写为JUC

这个包内包含所有的与并发相关的操作，因此取名为JUC

>并发：cpu快速切换程序执行，形成同时运行的假象（一段时间内程序同时运行）
>
>并行：相对于串行而言，指多个程序同时执行（同一时刻程序同时运行）

## 守护线程

> 守护线程（也叫后台线程）：
>
> 为用户线程提供公共服务，没有用户线程时会自动离开

特点：

- 优先级比较低
- 普通线程可以通过`setDaemon(true)`来设置一个线程为守护线程
- 守护线程中**创建的新线程依然是守护线程**
- **守护线程是JVM级别的**；以 Tomcat 为例，如果你在 Web 应用中启动一个线程，这个线程的 生命周期并不会和 Web 应用程序保持同步。也就是说，即使你停止了 Web 应用，这个线程依旧是活跃的
- 只要有一个用户线程，那么守护线程就不会退出；如果全是守护线程，那么守护线程也就会退出

Java默认有两个线程：

1. `main`线程
2. `GC`线程

其中GC线程就是守护线程，当GC线程是JVM中仅剩的线程时，GC线程会自动离开

## 线程池

#### 线程池的作用

1. **增快响应速度**
2. **控制并发量**（最主要的原因）
3. **对线程进行统一管理**
4. **减小线程切换时的上下文开销**

> 实现原理：每一个Thread都有一个start方法，当调用start启动线程时，JVM就会调用该类的run方法
>
> **线程池就是通过不断向start方法中传递Runnable对象**

#### 线程池的组成&参数

1. 线程池管理器：用于创建并管理线程池 
2. 工作线程：线程池中的线程
3. 任务接口：每个任务必须实现的接口，用于工作线程调度其运行 
4. 任务队列：用于存放待处理的任务，提供一种缓冲机制

在`Executor`框架内，`ThreadPoolExecutor`负责创建线程池，构造方法如下：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```

- `corePoolSize`：线程池线程数量
- `maximumPoolSize`：最大线程数量
- `keepAliveTime`：最大连接时长（**当前线程数量处于上面两个数量之间，就会判断最大连接时长**）
- `unit`：时间单位
- `workQueue`：阻塞队列，被提交但是没有被执行的任务
- `threadFactory`：线程工厂，这里使用默认的线程工厂
- `handler`：拒绝策略

#### 线程池的状态

`ThreadPoolExecutor`有五种状态：这五个状态由`ctl`来控制，`ctl`是一个`AtomicInteger`类型的变量，状态就由`ctl`来获取

```java
private static final int RUNNING    = -1 << COUNT_BITS;
// 线程池创建后处于Running状态
private static final int SHUTDOWN   =  0 << COUNT_BITS;
// 调用shutdown方法进入，不能接受新的任务，但是会将阻塞队列中的任务执行完毕
private static final int STOP       =  1 << COUNT_BITS;
// 调用shutDownNow进入STOP状态，线程池不能接受新的任务，阻塞队列中的任务也会被丢弃
private static final int TIDYING    =  2 << COUNT_BITS;
// 所有任务终止，ctl记录的任务数量为0，就会变为TIDYING（接着会执行Terminated()函数）
private static final int TERMINATED =  3 << COUNT_BITS;
// 执行完terminated方法后，就会由TIDYING转变为TERMINATED状态
```

#### 拒绝策略

> 线程池中线程已经使用完，且任务队列也已经满了，此时就需要对新来的任务进行拒绝

JDK内置有四种拒绝策略，这四种拒绝策略是`ThreadPoolExecutor`类的内部类

1. `AbortPolicy` ：**直接抛出异常**，阻止系统正常运行。 
2. `CallerRunsPolicy`： 只要线程池未关闭，该策略**直接在调用者线程中，运行当前被丢弃的任务**。（显然这样做不会真的丢弃任务，但是，任务提交线程的性能极有可能会急剧下降）
3. `DiscardOldestPolicy` ： **丢弃最老的一个请求**，也就是即将被执行的一个任务，并尝试再次提交当前任务
4. `DiscardPolicy`： 该策略默默地丢弃无法处理的任务，不予任何处理。如果允许任务丢失，这是最好的一种方案。

### 不同线程池

----

Java中有四种线程池，他们的顶层接口是`Executor`，但是严格意义上来说`Executor`并不是一个线程池，而是一个执行线程池的工具，**真正的线程池接口是`ExecutorService`**

`ExecutorService`有四个静态方法：

- `newSingleThreadExecutor`
- `newFixedThreadPool`
- `newScheduledThreadPool`
- `newCachedThreadPool`

下面我们来说这些不同线程池的特点

#### newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

特点：

- 只有一个
- 所有任务按照**先来先执行**的顺序执行

#### newScheduledThreadPool

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

//ScheduledThreadPoolExecutor():
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
```

特点：

- 可以定时执行

`schedule()`方法传入三个参数：

1. `Runnable`接口
2. 延迟时间
3. 时间单位

`scheduleAtFixedRate()`有四个参数：

1. `Runnable`接口
2. 初始延迟时间
3. 执行周期
4. 时间单位

demo：

```java
public static void main(String[] args) {
    ScheduledExecutorService ses = Executors.newScheduledThreadPool(3);
    ses.schedule(()-> System.out.println("延迟3s后执行"),3, TimeUnit.SECONDS);
    ses.scheduleAtFixedRate(()-> System.out.println("最开始延迟5s后，每三秒执行一次"),5,3, TimeUnit.SECONDS);
    ses.shutdownNow();
}
```

#### newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

特点：

- 没有创建核心线程（核心线程数为0），最大线程数为`Integer.MAX_VALUE`
- 将任务添加到**同步等待队列**`SynchronousQueue`（如果入列成功，那么会等待空闲的线程去运行，如果没有空闲线程，会创建线程运行）

- 适用于**短期异步程序**
- 若一个线程**60s**未被使用，会被移除

#### newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
```

特点：

- 创建有n个线程的线程池
- **只会创建核心线程！**（**因为核心线程数与非核心线程数相等**）
- 如果任务队列没有任务，线程会阻塞在`take`方法，**不会被回收**
- 如果线程因失败或异常而终止，那么会创建一个新线程代替他持续后续的任务（可选）
- 池若不关闭，线程也不会移除

### 线程池工作原理

![Executor框架](http://img.yesmylord.cn//Executor.png)

由图可以看出，创建线程池的是`Executors`类，回到第一节的demo

总体的运行过程如下：

1. 线程池刚创建时，**内部没有一个线程**
2. 当调用`execute()`方法添加任务，会与`corePoolSize`进行对比
   - 如果正在运行的线程数量小于`corePoolSize`，马上创建线程运行这个任务
   - 如果正在运行的线程数量大于等于`corePoolSize`，那么这个任务放入**任务队列**
   - 如果任务队列满了，而且正在运行的线程数量小于`maxmumPoolSize`，那么还是要创建非核心线程立刻运行这个任务
   - 如果任务队列满了，而且正在运行的线程数量大于等于`maxmumPoolSize`，那么会抛出`RejectExecutionException`异常（默认的抛弃策略`AbortPolicy`）
3. 线程完成任务会从任务队列找下一个任务来执行
4. 当一个线程闲置，并且运行时间超过`keepAliveTime`时，线程池会判断，如果当前运行的线程数量大于`corePoolSize`，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到`corePoolSize`的大小

原理如图：

![线程池工作原理](http://img.yesmylord.cn//image-20210809171055450.png)

## 阻塞队列

`BolckingQueue`的API：

| 方法\处理方式 | 抛出异常  | 返回特殊值 |  一直阻塞  |      超时退出      |
| :-----------: | :-------: | :--------: | :--------: | :----------------: |
|   插入方法    |  add(e)   |  offer(e)  | **put(e)** | offer(e,time,unit) |
|   移除方法    | remove()  |   poll()   | **take()** |  poll(time,unit)   |
|   检查方法    | element() |   peek()   |     -      |         -          |

常用的实现了此接口的类有：

- `ArrayBlockingQueue`
  - 底层由数组组成，**有界**的阻塞队列
  - 可以指定初始化大小，一旦初始化不能修改
  - 构造方法中可以设置是否为公平锁
- `LinkedBlockingQueue`
  - 无界的阻塞队列
  - 底层是链表
  - 队列按照**先进先出**的原则对元素进行排序
- `DelayQueue`
  - 该队列中的元素**只有当其指定的延迟时间到了，才能够从队列中获取到该元素**
  - 也没有大小限制
- `PriorityBlockingQueue`
  - 基于优先级的无界阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定）
  - 内部控制线程同步的锁采用的是**非公平锁**
- `SynchronousQueue`
  - 比较特殊，没有容器存储
  - 一个`put`必须等一个`take`，一个`take`必须等一个`put`

## Java中线程的方法与状态转换

在JDK源码中，`Thread.State`类代码如下，有**六个状态**：

```java
public enum State {
	// 新建
    NEW,
    // 运行
    RUNNABLE,
    // 阻塞
    BLOCKED,
    //等待
    WAITING,
    //超时等待
    TIMED_WAITING,
   	// 终止状态
    TERMINATED;
}
```

线程的基本方法有`wait()`、`notify()`、`notifyAll()`、`sleep()`、`join()`、`yield`

![Java线程方法与状态变化图](http://img.yesmylord.cn//image-20210803204224306.png)

- `wait()`：直接调用后会进入`waiting`状态；会释放锁；加时间参数的话，会进入`TIMED_WAITING`状态

  - **注意：**`wait()`方法不能写在`if`的执行语句中，如果有此需求，可以使用`while`进行判断（**虚假唤醒**）

- `notify()`：唤醒在一个锁上等待的**单个**线程；如果有很多线程，会随机选择一个唤醒

- `sleep()`：进入`TIMED_WAITING`状态，不会释放当前占有的锁；

- `yield()`：会让线程从执行进入就绪状态，让出当前CPU时间片

- `interrupt()`：本意是给这个线程一个通知信号，会影响这个线程内部的一个中断标示位；**不会改变线程的状态**

  1. 调用方法不会中断一个正在运行的线程；仅仅只是改变了一个中断标识位

  2. 若线程原本调用`sleep()`而处于`TIMED_WAITING`状态，调用此方法会抛出`InterruptException`，从而使线程提前结束`TIMED_WAITING`状态

  3. 抛出`InterruptException`后，会恢复中断标志位

  4. 中断状态是线程固有的一个标识位，可以通过此标识位安全的终止线程

     比如，你想终止 一个 `thread`时，可以调用`thread.interrupt()`方法，在线程的 `run` 方法内部可以根据`thread.isInterrupted()`的值来优雅的终止线程

- `join()`：**等待其他线程终止**，在当前线程中调用`join()`，会使当前线程阻塞，等到另一个线程结束，才会变为就绪状态。

  - 为什么要有`join()`方法？很多情况下主线程启动了子线程，需要用到子线程的返回结果，即主线需要等到子线程结束后再结束，就有了`join`方法

---

### 虚假唤醒

> 虚假唤醒：例如，生产者生产了1个商品，但是却唤醒了3个消费者来消费，最终只能有一个消费者消费成功，其他两个线程就被“忽悠”了

测试demo：

```java
class PV{
    private int x = 0;
    public synchronized void p() throws InterruptedException {
        if( x == 0){// 将这里改为while
            this.wait();
        }
        x --;
        System.out.println(Thread.currentThread().getName()+"："+x);
        this.notifyAll();
    }
    public synchronized void v() throws InterruptedException {
        if( x != 0){// 将这里改为while
            this.wait();
        }
        x ++;
        System.out.println(Thread.currentThread().getName()+"："+x);
        this.notifyAll();
    }
}
```

main方法

```java
public static void main(String[] args) {
    PV pv = new PV();
    new Thread(()-> {
        for (int i = 0; i < 10; i++) {
            try {
                pv.v();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    },"A").start();
    // 同上，继续创建线程 B、C、D分别运行 p()、v()、p()操作
}
```

运行结果如下：

```java
A：1
B：0
A：1
C：0
A：1
B：0
A：1
C：0
B：-1
B：-2
B：-3
C：-4
C：-5
...
```

发现出现了负数这种情况，显然不是我们想要的

为什么会出现这种问题？

注意这里

```java
if( x != 0){
    this.wait();
}
x ++;
this.notifyAll();
```

我们使用`if`进行判断，只会执行一次，如果该线程被唤醒，那么将不会去判断`x != 0`这个条件

所以要使用`while`，将方法中`if`判断改为while即可

总结：

如果要判断条件并进行`wait()`方法，不能使用`if()`，会出现虚假唤醒的现象

## 锁及相关概念（重点）

### 乐观锁与悲观锁

乐观锁与悲观锁是一种对于锁的思想：

- 乐观锁：认为**写入少**
- 悲观锁：认为**写入多**

由两种观点，就有不同的实现：

乐观锁认为写入少，所以**不会上锁**，但是更新时会进行一个判断（CAS操作），这样即使没有上锁，也不会出现线程安全问题

悲观锁认为写入多，在**每次读/写数据时都会进行上锁**，其他线程想要进行读写数据，必须先拿到锁（`Synchronized`就是悲观锁的一种实现）

### 什么是CAS

**CAS（Compare And Swap/Set）：比较并变换**，是一个**原子**操作，**相同则更新**

> `CAS(V,E,N)`
>
> V 表示要更新的变量（内存值）
>
> E 表示预期值（旧的）
>
> N 表示新值

1. 如果`V==E`值时，会将 `V=N`（内存值 == 预期值，说明没有线程对当前变量进行写操作）
2. 如果`V!=E`，则当前线程什么都不做（内存值 != 预期值，说明已经有其他线程做了更新，那么现在就不能更改这个值）
3. 最后，CAS操作返回当前 V 的真实值

注意：

- CAS 操作是**抱着乐观的态度**进行的（乐观锁），它总是认为自己可以成功完成操作
- CAS是一个自旋锁，会有一个线程进行自旋，反复判断是否符合条件
- 当多个线程同时使用 CAS 操作一个变量时，**只有一个会胜出，并成功更新**，其余均会失败
- **失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试**，当然也允许失败的线程放弃操作
- 基于这样的原理，**CAS 操作即使没有锁，也可以发现其他线程对当前线程的干扰**，并进行恰当的处理。

### ABA问题

CAS过程中，有ABA问题

![ABA示意图](http://img.yesmylord.cn//image-20210814202529921.png)

如何解决？

加一个版本号即可，每次更改这个值就对齐加1，然后cas比较这个版本号就知道是否出现了ABA问题

### 自旋锁

> 自旋锁：CPU对线程进行轮询，反复询问是否释放锁，直到释放为止
>
> 自旋周期：CPU轮询的时间

优点：减少了线程阻塞；对于锁的竞争不激烈，且占用锁时间非常短的代码块来说性能能大幅度上升

缺点：如果锁竞争激烈或是占用锁时间长，那么会持续的占用CPU是极大的性能损耗

在Java中，1.5时自旋周期时定死的，在1.6后加入了**适应性自旋锁**，由前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定

```java
// 自旋锁的开启
JDK1.6 中-XX:+UseSpinning 开启
-XX:PreBlockSpin=10 设置自旋次数
JDK1.7 后，去掉此参数，由 jvm 控制
```

### 可重入锁（递归锁）与不可重入锁

> 可重入锁（也叫递归锁）：
>
> 理解方式一：当一个线程获取对象锁之后，这个线程可以再次获取本对象上的锁，而其他的线程是不可以的
>
> 理解方式二：一个线程执行一个嵌套的方法时，当外部方法获取到锁，他内部调用的方法无需再去获取锁
>
> 理解方式三：锁分配的单位是线程，而不是方法。一个方法无论嵌套自身的方法多少次，锁依然在这个线程内，因此无需再获取锁

在JAVA环境下`ReentrantLock`和`synchronized`都是可重入锁

可重入锁的目的是为了解决死锁的问题

### 公平锁与非公平锁

公平锁（`Fair`）：加锁前检查是否有排队等待的线程，**优先排队等待的队列，先来先得**

非公平锁（`Nonfair`）：加锁不考虑排队等待问题，**直接尝试获取锁，获取不到自动到队尾等待**（可以插队）

注意：

- 非公平锁性能高于公平锁5-10倍，因为公平锁要维护等待队列
- 在Java中，`synchronized`是非公平锁，`ReentrantLock`**默认**的`lock()`方法采用的是非公平锁

### 共享锁和独占锁

独占锁：**每次只有一个线程能持有锁**；一种悲观策略，无论是读操作还是写操作，都会进行加锁。

共享锁：允许多个线程同时获取锁，并发访问，共享资源。一种乐观锁

注意：

- JUC中的`ReadWriteLock`读写锁，允许一个资源可以被**多个读操作**访问，或者被**一个写操作**访问，但两者不能同时进行

### AQS同步抽象队列

> AQS (AbstractQueuedSynchronizer)：抽象的队列式同步器
>
> ​		定义了一套多线程访问共享资源的同步器框架，很多锁都是通过AQS来实现的，例如`ReentrantLock、Semaphore、CountDownLatch`

这个抽象类主要维护了一个**状态state**还有一个**FIFO的线程等待队列**：

![AQS](http://img.yesmylord.cn//image-20210814200556108.png)

`state`状态：

```java
private volatile int state;
// state 代表共享资源；可以看到其使用volatile修饰
```

有三个方法可以操作这个状态的值：

```java
protected final int getState() {
    return state;
}

protected final void setState(int newState) {
    state = newState;
}

protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

这里贴一个demo

```java
public class AqsDemo implements Lock {
    private Sync sync = new Sync();

    // 建议使用内部类来实现
    private class Sync extends AbstractQueuedSynchronizer{
        @Override
        protected boolean tryAcquire(int arg) {
            assert arg == 1;
            if(compareAndSetState(0 ,1)){
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            assert arg == 1;
            // 判断当前线程是不是独一无二的持有当前锁
            if(!isHeldExclusively()) throw new IllegalMonitorStateException();
            // 如果一样，再去释放
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        @Override
        protected boolean isHeldExclusively() {
            // 获得当前的排他性线程，看看是不是当前线程
            return getExclusiveOwnerThread() == Thread.currentThread();
        }
    }

    @Override
    public void lock() {
        // 注意：调用的是acquire 方法，而不是自己实现的tryAcquire
        // 在Java内部帮你做了关于队列之间的操作，我们只需要关心逻辑
        sync.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {

    }

    @Override
    public boolean tryLock() {
        return false;
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return false;
    }

    @Override
    public void unlock() {
        sync.release(1);
    }

    @Override
    public Condition newCondition() {
        return null;
    }
}
```



### 锁升级

总共有四种：**无状态锁、偏向锁、轻量级锁、重量级锁**

在内存中，锁的信息**存放在对象头中的markword中**（markword包含的内容有三大部分：Hashcode、锁信息、GC信息）

> 锁升级：
>
> ​		随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级到重量级锁；
>
> ​		但是锁的升级只能是**单向**的，不存在降级

#### 偏向锁

大部分情况下锁不仅不存在多线程竞争，而且总是由同一线程多次获得

由此提出了偏向锁

> 偏向锁：在某个线程获得锁后，消除这个线程重入（CAS）的开销，看起来非常偏向这个线程，所以叫偏向锁

轻量级锁的获取及释放依赖多次CAS，但是偏向锁只需要在置换线程ID时依赖一次CAS指令

特点：

- **偏向锁主要用来优化同一线程多次申请同一个锁的竞争**

- 一次CAS，只比较Thread ID
- 消除了重入开销

#### 轻量级锁

> “轻量级”是相对于使用操作系统互斥量来实现的传统锁而言的

作用：

- 轻量级锁所适应的场景是**线程交替执行同步块**的情况，如果存在同一时间访问同一锁的情况，就会导致**轻量级锁膨胀为重量级锁**。
- 多次CAS，自旋判断

#### 重量级锁

`synchronized`是通过对象内部的一个叫做**监视器锁**来实现的，但是监视器锁本质又是依赖于操作系统底层的`Mutex lock`来实现的

操作系统想要实现一个重量级锁，必须从用户态切换到核心态，所以这也是`synchronized`效率低的原因

> 重量级锁：**依赖于操作系统的`Mutex Lock`实现的锁**

**JDK1.6 以后**，为了减少获得锁和释放锁所带来的性能消耗，提高性能，**引入了“轻量级锁”和 “偏向锁”**

#### 锁升级过程

![图源自敖丙博客](http://img.yesmylord.cn//image-20210818091932328.png)

当一个线程要去获取一个锁的时候，简单过程如下：

1. 检查是否是同一个线程（偏向锁）
   - 是：直接获得锁（可重入）
   - 否：继续向下执行
2. 升为轻量级锁
3. CAS操作
   - 获取到：获得锁
   - 获取不到：向下执行
4. **自旋锁**
5. 升级为重量级锁

### Synchronized

> Java中使用专门的关键字`Synchronized`，是**悲观锁**，也是**可重入锁**

直接修饰：

- **修饰方法**：锁住对象的实例(`this`)，即方法的调用者
- **修饰静态方法**：锁住`Class`实例（因为`Class`数据存放在永久代（元空间），此位置是全局共享的，所以相当于一个**全局锁**）
- **修饰对象**：锁住所有以该对象为锁的代码块。他有多个队列，当多个线程一起访问某个对象监视器的时候，对象监视器会将这些线程存储在不同的容器中

`synchronized(obj){}`同步块中

- `obj`称为**同步监视器**；可以是任何对象，但是推荐使用共享资源作为同步监视器（修饰方法时，同步监视器就是`this`或是`class`）

#### 底层实现

对象头的markword会关联到一个**monitor对象**（这个对象是用C++语言写的）

- 当我们进入一个方法的时候，执行**monitor enter**，就会获取当前对象的一个所有权，这个时候monitor进入数为1，当前的这个线程就是这个monitor的owner。
- 如果你已经是这个monitor的owner了，你再次进入，就会把进入数+1（每次重入加一）
- 同理，当他执行完**monitor exit**，对应的进入数就-1，直到为0，才可以被其他线程持有。

所有的互斥，其实在这里，就是看你能否获得monitor的所有权，一旦你成为owner就是获得者。

### Lock

`synchronized`是悲观锁，无论线程是读还是写都会独占整个资源，因此出现了`Lock`接口

JUC下有`locks`包，这个包内，最常见就有`ReentrantLock`与`ReentrantReadWriteLock`

![locks包的部分关系](http://img.yesmylord.cn//image-20210803181741298.png)

`ReentrantReadWriteLock`虽然没有实现`Lock`接口，但是他的两个静态内部类`ReadLock`与`WriteLock`均实现了`Lock`接口

`Lock`接口部分方法如下：

```java
void lock(); //若锁处于空闲状态，当前线程将获取到锁
boolean tryLock();//如果锁可用, 则获取锁, 并立即返回 true, 否则返回 false
/* tryLock()和lock()的区别在于：
	tryLock()只是"试图"获取锁, 如果锁不可用, 不会导致当前线程等待, 当前线程仍然继续往下执行代码. 
	lock()方法则是一定要获取到锁, 如果锁不可用, 就一直等待, 在未获得锁之前,当前线程并不继续向下执行.
*/
void unlock();
//执行此方法时, 当前线程将释放持有的锁. 锁只能由持有者释放, 如果线程并不持有锁, 却执行该方法, 可能导致异常的发生
Condition newCondition();
//条件对象，获取等待通知组件。该组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的 await()方法，而调用后，当前线程将缩放锁。
void lockInterruptibly();//使用此方法获取锁时，如果线程正在等待获取锁，那么这个线程可以响应中断，即可以中断线程的等待状态
/*也就是说，
	当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，
	假若此时线程A获取到了锁，而线程B只有在等待，
	那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。
*/
```

注意：

- **当一个线程获取了锁之后（运行状态），是不会被`interrupt()`方法中断的**；除非调用的是`lockInterruptibly()`方法获取锁

- **中断只能作用于处于WAITING状态的线程**

因此使用锁的基本方式均为：

```java
Lock lock = ...; // 声明一个锁
lock.lock();//加锁
try {
    // 同步操作
} catch (Exception e){
    e.printStackTrace();
}finally {
    lock.unlock(); 
    // 必须在finally中释放锁；因为lock即使发生异常也不会自动释放锁
}
```

### ReentrantLock

`ReentrantLock`继承了`Lock`接口并实现了接口中定义的方法，是一种**可重入锁**

![locks包的部分关系](http://img.yesmylord.cn//image-20210803181741298.png)

方法介绍：

![locks方法](http://img.yesmylord.cn//Package locks.png)

首先是实现了`Lock`接口的方法（上面已经介绍），其他是`ReentrantLock`自己的方法：

```java
getHoldCount(); //查询当前线程保持此锁的次数，也就是执行此线程执行 lock 方法的次数。
getQueueLength(); //返回正等待获取此锁的线程估计数，比如启动 10 个线程，1 个线程获得锁，此时返回的是 9
getWaitQueueLength(Condition condition); //返回等待与此锁相关的给定条件的线程估计数。
/* 比如 10 个线程，用同一个 condition 对象，并且此时这 10 个线程都执行了condition 对象的 await() 方法，那么此时执行此方法返回 10 */
hasWaiters(Condition condition);// 查询是否有线程等待与此锁有关的给定条件(condition)，对于指定 contidion 对象，有多少线程执行了 condition.await 方法
hasQueuedThread(Thread thread); // 查询给定线程是否等待获取此锁
hasQueuedThreads(); //是否有线程等待此锁
isFair(); //该锁是否公平锁
isHeldByCurrentThread();// 当前线程是否保持锁锁定，线程的执行 lock 方法的前后分别是 false 和 true
isLock();//此锁是否有任意线程占用
```

这里可以对比一下`synchronized`与`ReentrantLock`：

| 对比项       | synchronized  | ReentrantLock       |
| ------------ | ------------- | ------------------- |
| 如何加锁解锁 | JVM自动控制   | 程序员手动进行      |
| 是否公平     | 非公平锁      | 默认为非公平锁      |
| 是否可重入   | 可重入        | 可重入              |
| 发生异常     | JVM自动释放锁 | finally中手动释放锁 |
| 可中断锁     | 不可中断锁    | 可中断锁            |

总结：`ReentrantLock`对比`synchronized`主要增加了三项功能：

1. 等待可中断：当持有锁的线程长期不释放锁时，**正在等待的线程可以选择放弃等待**，改为处理其他事情，它对处理执行时间非常长的同步块很有帮助（而在等待由`synchronized`产生的互斥锁时，会一直**阻塞**，是不能被中断的）
2. 可实现公平锁：可以使用`new ReentrantLock(true)`来使用公平锁
3. 锁可以绑定多个条件：`ReentrantLock`对象可以同时绑定多个`Condition`对象（条件变量或条件队列）；而在`synchronized`中，锁对象的`wait()`和`notify()`或`notifyAll()`方法可以实现一个隐含条件，但如果要和多于一个的条件关联的时候，就不得不额外地添加一个锁，而`ReentrantLock`则无需这么做，只需要多次调用`newCondition()`方法即可。而且我们还可以通过绑定`Condition`对象来判断当前线程通知的是哪些线程（即与`Condition`对象绑定在一起的其它线程）

demo：

```java
public class Test {
    private ArrayList<Integer> arrayList = new ArrayList<Integer>();
    private Lock lock = new ReentrantLock();
    //注意这个地方，锁对象要放在成员变量这个地方
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
    }  
     
    public void insert(Thread thread) {
        // 锁的创建不能放在方法内，要不然每一个线程获得的都是不同的锁
        lock.lock();
        try {
            System.out.println(thread.getName()+"得到了锁");
            for(int i=0;i<5;i++) {
                arrayList.add(i);
            }
        } catch (Exception e) {
            // TODO: handle exception
        }finally {
            System.out.println(thread.getName()+"释放了锁");
            lock.unlock();
        }
    }
}
```

### Semaphore

> Semaphore：信号量，是对具体物理资源的抽象
>
> 处理多个共享资源的问题
>
> 关于信号量的详细解释，可以看我的另一篇[blog](https://www.yesmylord.cn/2020/12/09/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E8%BF%9B%E7%A8%8B/)

注意：现有资源数目信号量S。P、V操作分别代表消费者（申请资源）、生产者（释放资源）

- `S == 1`：信号量就变为互斥信号量
- `S > 0`：说明S资源还有S个
- `S < 0`：说明等待队列还有-S个进程阻塞着

Java中demo：

```java
// 信号量值为 3
Semaphore s = new Semaphore(3);
try {
    s.acquire();
    // 省略业务逻辑
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    s.release();
}
```

可见信号量与`ReentrantLock`使用该方法基本一致

### ReadWriteLock

读写锁将读操作与写操作进行分离：

```java
public interface ReadWriteLock {
    Lock readLock();// 返回读锁

    Lock writeLock();// 返回写锁
}
```

### 锁优化

有了锁虽然解决了线程安全问题，但是带来了性能的下降，此时就要进行锁优化了。

一般我们会有如下的锁优化方法：

- **减少锁持有时间**：只在有线程安全问题的程序上加锁
- **减小锁粒度**：将大对象拆成小对象，降低锁竞争
- **锁分离**：根据功能分离锁，例如`ReadWriteLock`，将读与写进行分离
- **锁粗化**：通常情况下，为了保证多线程间的有效并发，会要求每个线程持有锁的时间尽量短。但是，凡事都有一个度，如果对同一个锁不停的进行请求、同步和释放，其本身也会消耗系统宝贵的资源，反而不利于性能的优化 
- **锁消除**：编辑器级别的事情，可以对没有共享需求的代码进行优化，直接消除锁，这些多半是程序员编码不规范引起的。

### Volatile关键字

> volatile本意是“易失的”，在计算机内代表，被这个关键字修饰的变量**不会被缓存**起来；
>
> 对于非volatile变量来说，访问它的值会先从内存copy到CPU cache中，如果刚copy完，内存中的值就发生了改变，那么CPU读到的是cache中的值，而不是最新值
>
> 对于volatile修饰的变量，每次都要去内存中读取

被这个关键字修饰的变量代表着两种特性：**可见性与有序性**

- **变量可见性**：变量对所有线程可见（这里的可见性指：一个线程修改了变量的值，那么**新的值对于其他线程是可以立即获取的**）
- **禁止重排序**：多核CPU会对指令进行重排序，以加快指令的执行速度，使用此关键字可以不让CPU这么做

**优点：**

比`synchronized`更轻量级的一个同步锁，不会使线程阻塞

volatile 适合这种场景：**一个变量**被多个线程共享，线程直接给这个变量赋值

**注意：**

- 被`volatile`修饰的变量可以保证**单次读/写操作**的原子性
- 不能保证`i++`这种操作的原子性，因为本质上其是两次操作 读+写
- 必须同时满足两个条件，才能保证线程安全：
  - 对变量的写操作不依赖于当前值（`i++`），或者说是单纯的变量赋值（类似`flag = true`，不是这种`a += 10`）
  - 该变量没有包含在具有其他变量的不变式中（**不同的 volatile 变量之间，不能互相依赖**）只有在状态真正独立于程序内其他内容时才能使用 `volatile`

### 可见性与有序性实现的底层原理

> 底层是如何确保volatile的可见性的？

通过**缓存一致性协议**：不同厂商有不同协议，这里以牙膏厂的MESI为例：

​		**当CPU写数据时**，**如果发现操作的变量是共享变量**，会**发出信号通知其他CPU将该变量的缓存行置为无效状态**

> 底层是如何确保volatile的有序性的？

通过**内存屏障**，这是一个CPU指令，不能对其进行重排序，volatile就是基于内存屏障实现的

## JUC通信工具类

| 类             | 作用                                       |
| -------------- | ------------------------------------------ |
| Semaphore      | 限制线程的数量                             |
| Exchanger      | 两个线程交换数据                           |
| CountDownLatch | 线程等待直到计数器减为0时开始工作          |
| CyclicBarrier  | 作用跟CountDownLatch类似，但是可以重复使用 |
| Phaser         | 增强的CyclicBarrier                        |

### Semaphore

**用于资源有限的场景中，可以限制线程的数量**

比如我想限制只有3个线程在工作：

```java
public class SemaphoreDemo {
    static class MyThread implements Runnable {

        private int value;
        private Semaphore semaphore;

        public MyThread(int value, Semaphore semaphore) {
            this.value = value;
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire(); // 获取permit
                System.out.println(
                        String.format(
                                "当前线程是%d, 还剩%d个资源，还有%d个线程在等待",
                                value,
                                semaphore.availablePermits(), semaphore.getQueueLength()
                        )
                );
                // 睡眠随机时间，打乱释放顺序
                Random random = new Random();
                Thread.sleep(random.nextInt(1000));
                System.out.println(String.format("线程%d释放了资源", value));
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release(); // 释放permit
            }
        }
    }

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        // 最多只可以有三个线程在工作
        for (int i = 0; i < 10; i++) {
            new Thread(new MyThread(i, semaphore)).start();
        }
    }
}
```

### Exchanger

**用于两个线程交换数据，数据支持泛型**（所以我们可以传IO流之类的）

调用到`exchange()`方法，线程会进入阻塞状态，只有另一个`exchange()`

方法被调用，才会继续执行

核心方法：

- `exchange(E e)`：将数据交给另一个线程（会进入阻塞）

```java
final static Exchanger<String> ex1 = new Exchanger<>();
public static void main(String[] args) {
    for (int i = 0; i < 4; i++) {
        new Thread(() -> {
            try {
                String msg1 = ex1.exchange(Thread.currentThread().getName() + "向你问好");
                System.out.println(Thread.currentThread().getName()+"收到信息：" + msg1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

### CountDownLatch

> 闭锁、或者叫门闩：在闭锁到达结束状态之前，这扇门一直是关闭的。可以用来等其他线程执行。

假设**某个任务执行之前，需要等待其他线程完成一些任务**，那么就可以用`CountDownLatch`类

主要的方法有：

- `new CountDownLatch(int count)`：构造方法，参数是一个`int`值，代表需要等待几个任务
- `await()`：进入等待状态
- `await(long time, TimeUnit unit)`：进入等待状态，如果count为0或者时间到也会释放
- `getCount()`：获取当前`count`值
- `countDown()`：让`count`值减1，如果`count`为0，就会自动解锁`await`

```java
public class CountDownLatchDemo {
    // 定义前置任务线程
    static class PreTaskThread implements Runnable {
        private String task;
        private CountDownLatch countDownLatch;

        public PreTaskThread(String task, CountDownLatch countDownLatch) {
            this.task = task;
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            try {
                Random random = new Random();
                Thread.sleep(random.nextInt(1000));
                System.out.println(task + " - 任务完成");
                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        // 假设有三个模块需要加载
        CountDownLatch countDownLatch = new CountDownLatch(3);

        // 主任务
        new Thread(() -> {
            try {
                System.out.println("等待数据加载...");
                System.out.println(String.format("还有%d个前置任务", countDownLatch.getCount()));
                countDownLatch.await();
                System.out.println("数据加载完成，正式开始游戏！");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // 前置任务
        new Thread(new PreTaskThread("加载地图数据", countDownLatch)).start();
        new Thread(new PreTaskThread("加载人物模型", countDownLatch)).start();
        new Thread(new PreTaskThread("加载背景音乐", countDownLatch)).start();
    }
}
```

### CyclicBarrier

> 栅栏，

`CyclicBarrirer`从名字上来理解是“循环的屏障”的意思。

前面提到了`CountDownLatch`一旦计数值`count`被降为0后，就不能再重新设置了，它**只能起一次“屏障”的作用**。

而`CyclicBarrier`拥有`CountDownLatch`的所有功能，还可以使用`reset()`方法重置屏障

```java
public class CyclicBarrierDemo {
    static class PreTaskThread implements Runnable {

        private String task;
        private CyclicBarrier cyclicBarrier;

        public PreTaskThread(String task, CyclicBarrier cyclicBarrier) {
            this.task = task;
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            // 假设总共三个关卡
            for (int i = 1; i < 4; i++) {
                try {
                    Random random = new Random();
                    Thread.sleep(random.nextInt(1000));
                    System.out.println(String.format("关卡%d的任务%s完成", i, task));
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                cyclicBarrier.reset(); 
                // 重置屏障
            }
        }
    }

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> {
            System.out.println("本关卡所有前置任务完成，开始游戏...");
        });

        new Thread(new PreTaskThread("加载地图数据", cyclicBarrier)).start();
        new Thread(new PreTaskThread("加载人物模型", cyclicBarrier)).start();
        new Thread(new PreTaskThread("加载背景音乐", cyclicBarrier)).start();
    }
}
```

## CopyOnWriteArrayList

并发编程时，使用`ArrayList`会遇到`Concurrent Modification Exception`并发修改异常，表明ArrayList不能在并发开发中使用

```java
public static void main(String[] args) {
    final List<Integer> list = new ArrayList<>();
    //      1. 使用vector List<Integer> list = new Vector<>();
    //      2. 使用 List<Integer> list = Collections.synchronizedList(new ArrayList<>());
    //      3. CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>();
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                list.add(i);
                System.out.println(list);
            }
        }
    };
    new Thread(runnable).start();
    new Thread(runnable).start();
    // ConcurrentModificationException 抛出
}
```

为了避免这个异常，我们有三种解决办法：

1. 使用`Vector`，这个类是线程安全的，但是效率极低
2. 使用集合类的`synchronizedList`方法
3. 使用`CopyOnWriteArrayList`，这是最佳的方法

CopyOnWrite：写入时复制（COW 计算机程序设计领域的一种优化策略）

## ThreadLocal

> ThreadLocal 线程本地变量：
>
> 提供一个线程内的局部变量，**减少**同一个线程内多个函数或者组件之间一些**公共变量的传递的复杂度**

作用：主要是实现了**数据隔离**

### ThreadLocal的使用

`ThreadLocal`的使用非常简单，例如这个demo

```java
ThreadLocal<String> threadLocal = new ThreadLocal<>();
threadLocal.set("存值");
String s = threadLocal.get();
threadLocal.remove();
```

主要使用的方法就四个：构造方法、`set`、`get`、`remove`

### ThreadLocal的原理

要想搞清楚`ThreadLocal`，首先看`Thread`类中含有两个属性均为`ThreadLocalMap`类型（这里先知道Thread类有这个属性即可）

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
// 初始值均为Null
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
// inheritableThreadLocals是为了实现父子线程间共享threadLocal数据而提供的
```

再来看`Thread`的构造方法，很简单，与默认构造一样：

```java
public ThreadLocal() {
}
```

在看`get()`方法：

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t); 
    // 此处getMap返回了当前线程的ThreadLocal的值，如果为null，说明没有初始化，那么就初始化一下
    if (map != null) {
        // 如果不为null，说明有ThreadLocalMap，就去取值，由getEntry实现了真正的取值
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    // 如果这个值为null，说明还没初始化，就初始化一下
    return setInitialValue();
    // 这个方法点到最后，就是通过 new 创建了 LocalThreadMap
}
// get 方法中初始化 ThreadLocal 对象源码如下：
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        // 创建LocalThreadMap
        createMap(t, value);
    return value;
}
//最终创建ThreadLocal对象的代码如下：
void createMap(Thread t, T firstValue) {
    // 这里new了一个ThreadLocal
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

真正的`get`其实是由这里的`getEntry()`方法完成的：值得一提的是在ThreadLocalMap中是使用**开放地址法**处理哈希碰撞的

> 处理哈希碰撞的方法：
>
> - **开放地址法**：如果遇到哈希冲突，就重新寻找真正的存放数据的下标位置（重新计算哈希也有不同的方法）
>   - 线性探测（ThreadLocalMap就是这种方式）：从此下标开始，挨个往下找
>   - 二次探测：探测步数是原始相隔位置的平方
>   - 再哈希法：用不同哈希函数再求一遍哈希值
> - **链地址法**：如果遇到哈希冲突，就拉一条链表出来。HashMap中就是使用这种方法进行处理的
> - **建立公共溢出区**：专门存放所有哈希碰撞后的数据

```java
private Entry getEntry(ThreadLocal<?> key) {
    // Entry对象 是ThreadLocalMap的一个对象，他类似于HashMap中的Entry，都是实际存储数据的位置
    int i = key.threadLocalHashCode & (table.length - 1);
   	// 获取哈希表中该值的下标
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        // 因为使用开放地址法，所以这里需要重新找下标
        return getEntryAfterMiss(key, i, e);
}


private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;// table是一个Entry数组
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
// 可以看到，寻找的方式是线性探索
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
```

（`set()`方法实现的原理与`get()`方法差不多，就不再赘述；）

Entry代码如下，注意继承了**弱引用**类：

> **弱引用** —— 发现即回收
>
> 特点：
>
> - 只被弱引用关联的对象**只能生存到下一次 GC 发生为止**（无论内存是否足够）
> - 由于垃圾回收线程的优先级很低，所以不一定很快被回收掉；这种情况可以存活较长时间

```java
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** 注意这里Entry继承了弱引用类*/
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            // 注意key是ThreadLocal本身！！
            super(k);
            value = v;
        }
    }
    ...省略代码...
}
```

这里会产生**内存泄露**的问题：`table`是一个Entry数组，当发生GC时，`Entry`中的`key`是弱引用的，发生GC时会被回收，如果创建`ThreadLocal`的线程一直持续运行，那么这个`Entry`对象中的`value`就有可能一直得不到回收，发生内存泄露（简单说就是：`key`会被直接回收，但是`value`不一定）

就比如线程池里面的线程，线程都是复用的，那么之前的线程实例处理完之后，出于复用的目的线程依然存活，所以，`ThreadLocal`设定的`value`值被持有，导致内存泄露。

所以这就是`remove`的作用，在使用完后，我们要调用`remove`清除数据

### 父子线程间共享数据

还记得Thread有一个属性`InheritableThreadLocal`吗，使用这个属性就可以实现**父子线程间的资源共享**

```java
private void test() {    
    final ThreadLocal threadLocal = new InheritableThreadLocal();       
    threadLocal.set("帅得一匹");    
    Thread t = new Thread() {        
        @Override        
        public void run() {            
            super.run();            
            Log.i( "张三帅么 =" + threadLocal.get());        
        }    
    };          
    t.start(); 
}
```



---

### 总结

- `ThreadLocalMap`是每个`Thread`都拥有的一个属性
- `ThreadLocalMap`是`ThreadLocal `线程的内部类
- 将一个共用的`ThreadLocal`静态实例作为`key`，将不同对象的引用保存到不同线程的`ThreadLocalMap`中，就可以使用这个`ThreadLocalMap`来获取不同线程的数据了
- `ThreadLocalMap`中Entry的key是`ThreadLocal`类，而且是弱引用（有内存泄露的风险）
- `ThreadLocalMap`中避免哈希碰撞的方法是**开放地址法 + 线性探索**
- `ThreadLocalMap`与`synchronized`都可以进行数据隔离，区别是ThreadLocal使用空间换时间，`synchronized`则相反

## 相关链接

1. [狂神说JUC编程](https://www.bilibili.com/video/BV1B7411L7tE?p=5)
2. [大佬猿人谷blog](https://yuanrengu.com/2020/7691e770.html)
3. [敖丙ThreadLocal](https://www.zhihu.com/question/341005993)
4. [我自己的博客](https://www.yesmylord.cn/2021/07/26/JVM/%E6%B7%B1%E5%85%A5Java%E8%99%9A%E6%8B%9F%E6%9C%BAGC%E7%AF%87/)
5. [敖丙锁升级](https://blog.csdn.net/qq_35190492/article/details/104691668?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162924889716780261918067%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=162924889716780261918067&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~hot_rank-1-104691668.first_rank_v2_pc_rank_v29&utm_term=aqs&spm=1018.2226.3001.4187)
6. [RedSpider社区](http://concurrent.redspider.group/article)

