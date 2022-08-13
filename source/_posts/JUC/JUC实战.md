---
title: JUC实战
date: 2021-09-04 19:37:20
tags:
- JUC
categories:
- JUC
---

<center>
  引言：JUC实战
</center>

<!-- more -->

# JUC实战

之前的JUC，感觉也只是入了门；

在大学不管是做项目、还是做课设，都没有涉及到多线程的开发。

所以这篇可以更深刻的理解，JUC的由来~

## 很重要的前置知识

并发编程主要就是三点：**分工、同步、互斥**

并发编程要解决的问题（微观）：**原子性、可见性、有序性**

并发编程要解决的问题（宏观）：**安全性、活跃性、性能**

Java如何解决三个问题：`volatile`、`synchronized`、`final`、八项`Happens-Before`、锁

![并发编程脑图](http://img.yesmylord.cn//image-20210904194900277.png)

### happens before原则

1. 程序顺序原则（线程内必须串行执行）
2. 锁规则（解锁必须发生在上锁后）
3. volatile规则（强迫每次的读写都必须刷新到主内存，不能为了省事直接去工作内存读）
4. 线程启动规则（线程的`start()`方法先于它的其他操作）
5. 传递性（A先于B，B先于C，A必先于C）
6. 线程终止规则
7. 线程中断规则（线程的所有操作先于线程的终结）
8. 对象终结规则（构造方法先于`finalize()`方法）

这八个规则确定的内容，即使没有锁等同步操作，也可以按序执行

### 锁模型

> 简易的锁模型

```java
lock();
// 临界区代码
unlock();
```

> 改进的锁模型：**锁和锁要保护的资源是有对应关系的**

```java
// 1、 创建保护资源R的锁 LR
lock(LR); //2、上锁
// 3、 临界区操作R
unlock(LR);//4、释放锁
```

### 对象头

> 加锁的本质，**就是在锁对象的对象头写入了当前线程的ID**，获得了对Monitor对象的所有权

对象的组成：三大部分

- **对象头**
  - **Markword**（8字节，64位JVM）
  - **类型指针**（4字节，64位JVM）
  - 数组长度（数组才有此字段）
- 实例数据
- 字节填充

Markword记录了三方面的信息：哈希值、GC信息、锁信息

[对象头的锁升级过程](https://blog.csdn.net/weixin_39908462/article/details/111656725)（细品这篇博客，讲到了锁升级的过程）

![32位JVM的对象头](http://img.yesmylord.cn//image-20210904204832937.png)

## synchronized

### 1. 一把锁保护一个资源

```java
// 一把锁保护一个资源的例子
public class ALockProtectAResource {
    long value = 0;
    // 对于long 和 double的读与写操作，JVM是分两步完成的，存在安全问题
    synchronized long getValue(){
        return value;
    }
    synchronized void addOne() {
        value++;
    }
    /* syn修饰不同的位置有不同的作用：
    * 1. 修饰 普通的方法 锁住的是对象的实例 即 this
    * 2. 修饰 静态方法，锁住的是对应的Class对象
    * 3. 修饰 同步代码块，锁住的是传入的锁
    * */
}
```

### 2. 多把锁保护多个没有关联的资源

场景：有`account`与`password`字段。

对于`account`可以取款，查账

对于`password`可以修改、查看密码

> 为什么不用syn修饰方法呢？这样不也可以同步吗？

是可以同步，但是发现没有，密码业务与账户业务没有关系。

如果给方法加了syn，就锁住了this，导致两个业务之间也变为互斥了！降低了我们系统的效率

```java
// 多把锁保护多个没关系的资源
public class LocksProtectNoRelatedResource {
    // 密码相关
    private String password;
    private final Object lockMyPass = new Object();
    // 加上final，告诉编译器这是一个不可变对象，尽情的去优化吧
    void changePassword(String newPassword){
        synchronized (lockMyPass){
            password = newPassword;
        }
    }
    String getPassword(){
        synchronized (lockMyPass){
            return password;
        }
    }
    // 账户相关
    private Integer account;
    private final Object lockMyMoney = new Object();
    // 加上final，告诉编译器这是一个不可变对象，尽情的去优化吧
    Integer getAccount(){
        synchronized (lockMyMoney){
            return account;
        }
    }
    void takeMoney(Integer money){
        synchronized (lockMyMoney){
            account-=money;
        }
    }
}
```

### 3. 多把锁保护多个有关联的资源

场景：转账业务

A的账户需要扣除钱，B的账户需要加上钱（这里A与B是两个资源，而且他们需要同时进行操作）

实现的核心就是，要保证同一个锁锁住临界区的操作

> 实现一：错误的示范

```java
// 实现一：这是有问题的实现
class Account1 {
    private int balance;
    // 转账虽然加了syn，锁住了this，但是！我们操作的过程中还操作了B的账户！
    synchronized void transfer(Account1 target, int amt) {
        if (this.balance > amt) {
            this.balance -= amt;
            target.balance += amt;
        }
    }
}
```

> 实现二：必须要传相同的锁

```java
// 实现二：可以实现，但是有点问题
class Account2 {
    private int balance;
    private Object lock;
    // 构造时传入一个对象作为锁
    Account2(Object lock){
        this.lock = lock;
    }

    void transfer(Account2 target, int amt) {
        // 改变锁的对象，只要A与B两个人构造时传入相同的对象就可以了
        // 但是怕就怕两个人传入的锁不同
        synchronized (lock) {
            if (this.balance > amt) {
                this.balance -= amt;
                target.balance += amt;
            }
        }
    }
}
```

> 实现三：直接用class对象当锁

这种实现也有问题，就是性能不高；

A转B、C转D，这两个不需要互斥的操作在这种实现下也变得互斥了

```java
// 实现三
class Account3 {
    private int balance;

    void transfer(Account3 target, int amt) {
        // 改变锁的对象，直接传Class对象
        synchronized (Account3.class){
            if (this.balance > amt) {
                this.balance -= amt;
                target.balance += amt;
            }
        }
    }
}
```

> 实现四：使用N把锁，操作时必须同时取到

对于这个场景：完全可以锁住`this`与`target`两个对象

但是存在**死锁**问题，设想，A在给B转账的同时，B也在给A转账（看代码中标有记号的位置）	

```java
// 实现四：使用两把锁，进行两次判断
class Account4 {
    private int balance;

    void transfer(Account4 target, int amt) {
        synchronized (this){
            // #这里会死锁#：A执行到这里，B也执行到了这里
            // 由于A想拿B的this，B也想拿A的this，导致双方都不能继续进行下去
            synchronized (target){
                if (this.balance > amt) {
                    this.balance -= amt;
                    target.balance += amt;
                }
            }
        }
    }
}
```

## 死锁

上一节的实现四，出现了死锁问题

死锁部分可以看我的另一篇[blog](https://www.yesmylord.cn/2020/12/20/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%AD%BB%E9%94%81/#more)

### 死锁产生的必要条件

1. **互斥条件**：进程对其所要求的资源进行排它性控制，即一次只有一个进程可以使用一个资源。
2. **请求和保持条件**：进程已经保持了至少一个资源，但又提出了新的资源请求。
3. **不可剥夺条件**：进程所获得的资源在未被释放之前，不能被其它进程强行剥夺。
4. **环路条件**：在发生死锁时，必然存在一个进程资源的循环等待链

其中互斥条件不能被破坏，其他三个都是可以破坏的

### 破坏请求和保持条件：有两种方案

1、可以将进程所需的所有资源一次性拿走（但是会导致资源浪费、饥饿问题产生）

2、只获得初期所需资源后，开始运行。运行过程逐步释放已分配、已用完的全部资源，再请求新的所需资源

对于转账这个业务，第二种方案不好实现，但是第一种方案还是可以实现的

```java
// 实现5，破坏请求和保持条件
// 额外的一个类，帮我们申请资源，防止死锁
class Allocater{
    // Allocater要保持单例模式，这里使用了饿汉式单例
    private Allocater(){}
    private final static Allocater allocater = new Allocater();
    private List<Object> als = new ArrayList<>();

    public static Allocater getAllocater(){
        return allocater;
    }
	// 申请
    synchronized boolean apply(Object a, Object b){
        if(als.contains(a) || als.contains(b)){
            return false;
        }else {
            als.add(a);
            als.add(b);
            return true;
        }
    }
	// 释放
    synchronized void free(Object a, Object b){
        als.remove(a);
        als.remove(b);
    }
}
class Account5{
    private int balance;

    void transfer(Account5 target, int amt) {
        Allocater allocater = Allocater.getAllocater();
        while (!allocater.apply(this, target));
        // 死循环，保证可以拿到两个资源
        try{
            synchronized (this){
                synchronized (target){
                    if (this.balance > amt) {
                        this.balance -= amt;
                        target.balance += amt;
                    }
                }
            }
        }finally {
            allocater.free(this, target);
        }
    }
}
```

### 破坏不可剥夺条件

Syn做不到破坏此项，因为Syn锁的申请与释放是JVM帮助我们管理的

但是Java中的Lock可以做到这一件事情，下面再讲

### 破坏环路条件

- 做法：系统**给每类资源赋予一个编号**，每一个进程按编号递增的顺序请求资源，释放则相反
- 编号的原则：较为紧缺的资源给以一个较大的序号
- 优点：较前两种策略，资源利用率和系统吞吐量，都有显著的改善。
- 问题：
  - 限制了新设备类型的增加
  - 发生作业使用资源的顺序与系统规定顺序不同的情况，造成资源的浪费，如：某进程先用磁带机，后用打印机，但按系统规定，它应先申请打印机，后申请磁带机，致使打印机长期闲置
  - 限制了用户简单、自由的编程

对于这个场景也很简单，给Account加一个id，用来排序

如果同时出现A转账B，B转账A的情况，由于id小的先申请，所以他们同时先申请同一个资源，不会出现环路，也就避免了死锁。

```java
// 实现6，破坏环路条件，给资源排序
class Account6 {
    private int balance;
    private int id;

    void transfer(Account6 target, int amt) {
        Account6 first = this;
        Account6 second = target;
        // 序号小的先申请
        if(target.id < this.id){
            first = target;
            second = this;
        }
        synchronized (first){
            synchronized (second){
                if (this.balance > amt) {
                    this.balance -= amt;
                    target.balance += amt;
                }
            }
        }
    }
}
```

## wait-notify 等待通知机制

在上面我们解决死锁的时候，使用了

```java
while (!allocater.apply(this, target));
```

死循环，让CPU自旋，来保证拿到资源，但是这样太耗费CPU了

wait-notify等待通知是更优的一种方案

#### Synchronized与wait-notify配合

首先来说明一下api吧：

他们都是`Obejct`类的方法

- `wait()`：将当前线程移入等待队列
- `notify()`：**随机唤醒**一个等待队列中的一个线程
- `notifyAll()`：**唤醒**等待队列中的**所有**线程

> 注意：尽量使用`notifyAll`！好像`notify`只唤醒一个线程，是不是会更安全一点呢？但这只是你自己的想象

假如这种情况：

有资源 A、B、C、D：

​		线程 1 申请到了 AB；线程 2 申请到了 CD；

​		此时线程 3 申 请 AB，会进入等待队列；

​		线程 4 申请 CD 也会进入等待队列；

现在我们再假设之后线程 1 归还了资源 AB

​		如果使用`notify()`来通知 等待队列中的线程，有可能被通知的是线程 4，但线程 4 申请的是 CD，所以此时线程 4 还 是会继续等待，而真正该唤醒的线程 3 就再也没有机会被唤醒了。

​		所以尽量使用`notifyAll()`

---

![wait-notify原理](http://img.yesmylord.cn//image-20210905110614638.png)

实现代码如下：

```java
class AllocaterNew {
    // Allocater要保持单例模式，这里使用了饿汉式单例
    private AllocaterNew(){}
    private final static AllocaterNew allocater = new AllocaterNew();
    private List<Object> als = new ArrayList<>();

    public static AllocaterNew getAllocater(){
        return allocater;
    }

    synchronized void apply(Object a, Object b){
        while (als.contains(a) || als.contains(b)) {
            // 不满足条件，进入等待队列
            try {
                wait();
            } catch (InterruptedException e) {
            }
        }
        als.add(a);
        als.add(b);
    }

    synchronized void free(Object a, Object b){
        als.remove(a);
        als.remove(b);
        notifyAll();// 唤醒所有线程
    }
}
```

## 安全性、活跃性、性能问题

### 安全性

即要保证线程安全，就得保证原子性、有序性、可见性

> 我们是不是每个对象都得分析它的三性？

只有一种情况我们需要分析：即分析 **可变的共享对象** 的原子性、安全性、可见性即可

---

此外有两个专业名词：

- **数据竞争**：指的就是可变的共享对象被抢来抢去
- **竞态条件**：程序的执行结果依赖于线程的执行顺序

### 活跃性

活跃性：其实也分了三个问题

- 死锁：前面提到了
- 活锁
- 饥饿

> 什么是活锁？

活锁就是，类似于线程之间都太客气了，互相谦让对方先使用资源

就和AB两个人进出同一个门一样，A靠右走让B，B靠左走让A，撞了上去

> 活锁怎么解决？

尝试**等待一个随机的时间**就可以了，简单但是很有效

> 什么是饥饿？

线程因无法访问所需资源而无法执行下去的情况

对于优先级低的线程，可能永远也得不到自己的资源，而无法执行

> 饥饿怎么解决？

有三种方案：

1. 保证资源充足
2. 避免持有锁的线程长时间进行
3. 公平的分配资源

其中1与2是比较难以实现的，资源不可能充足、持有锁的线程也很难缩短

所以只有公平的分配资源，比较好实现（类似于Java的公平锁）

### 性能问题

如果随意的使用锁，会导致性能急剧的下降

> 阿姆达尔定律：`S = 1 / ((1 - P) +  P / n )`
>
> n代表CPU核心线程数；
>
> P代表并行百分比；
>
> 1-P代表串行百分比；

![阿姆达尔定律](http://img.yesmylord.cn//image-20210905113012054.png)

假设我们的串行率(1-P)为5%，那么无论我们cpu有多少核心（n为无穷大）

S最终也只能为 20%

也就是说，如果串行率为5%，不管我们如何提高性能，最高也只能提高20%

> 如何提高性能？

- 使用无锁的数据结构与算法：比如ThreadLocal、CAS、COW、乐观锁
- 使用细粒度的锁：分段锁ConcurrentHashMap、读写锁ReadWriteLock

> 性能的指标：

1. 吞吐量：单位时间内能处理的请求数
2. 延迟：从发出请求到响应的时间
3. 并发量：能同时处理的请求数量

## 管程

`synchronized`的实现其实是**MESA管程模型**的简化版

而JUC包内，LOCK与Condition真正实现了MESA管程模型

> 管程是什么？

英文为Moniter、Java里面叫监视器（知道是啥了吧）

管程就是：**管理共享变量以及对共享变量的操作过程**，**让他们支持并发**

> 管程干了什么？

管程通过N个队列来**保证线程之间的互斥与同步**，入队出队操作由其封装

![MESA管程模型](http://img.yesmylord.cn//image-20210905173310258.png)

这种管程模型，条件可以有多个，但在Java的实现中，synchronized只有一个条件变量，也就是为什么说是简化版的`synchronized`

> 使用`wait`的正确姿势（这其实就是MESA模型规定的经典姿势）

```java
while(条件不满足){
	wait();
}
```

> `notify`如何使用？

如果你能确定以下三点，就可以使用`notify`，如果不能请使用`notifyAll()`

1. 所有线程都拥有相同的等待条件
2. 等待线程被唤醒后执行相同的操作
3. 只需要唤醒一个线程

## Java线程的状态转换

这个图绘制的很好

![Java线程方法与状态变化图](http://img.yesmylord.cn//image-20210803204224306.png)

注意：在OS层面，线程是有五个状态的（**新建、就绪、运行、阻塞、终止**）

但是JVM层面，将就绪与运行看做一个状态`RUNNABLE`（JVM不关心谁被调度了），而将阻塞分为三部分（`WAITING`、`TIMED_WAITING`、`BLOCKED`）

- `NEW`进入`RUNNABLE`：执行`start`方法
- 在OS内部：
  - 就绪进入运行状态：获得时间片
  - 运行进入就绪状态：`yield()`方法
- `RUNNABLE`与`WAITING`之间的状态转换：各有三种方式
  - `wait()`、`join()`、`LockSupport.park()`（`LockSupport`是Java中实现Lock的基础）
  - 状态反向：`notify()`、`notifyAll()`、`LockSupport.unpark(Thread thread)`
- `RUNNABLE`与`TIMED_WATING`状态的相互转换
  - 进入超时等待有五种方法`wait(long)`、 `join(long)` 、`sleep(long)`、`LockSupport.parkNanos(long)`、` LockSupport.parkUntil(long deadline)`
- `RUNNABLE`与`BLOCKED`的状态转换：
  - 只有一种方式：就是线程等待`synchronized`的锁

- 进入`TERMINATED`状态
  - 可以通过`stop`，但是这个方法已经不推荐使用了（Stop会立即杀了线程，但是锁不一定会释放（只会释放隐式锁））
  - 当线程 A 处于 `WAITING`、`TIMED_WAITING `状态时，如果其他线程调用线程 A 的 `interrupt()` 方法，会使线程 A 返回到 `RUNNABLE `状态，同时线程 A 的代码会触发 `InterruptedException` 异常，只要捕获这个异常我们就可以
  - 当线程A处于`RUNNABLE`状态时，可以同步不断的调用`isInterrupt()`方法，来判断自己是不是被别人叫停了

## Semaphore

`Semaphore`信号量，主要的`api`有：

- `new Semaphore(int permits , [boolean fair])`：创建一个信号量，permits代表资源的数量，fair代表创建一个公平锁还是非公平锁，默认为非公平
- `acquire()`：会将资源数 -1。如果为0，那么会进入等待状态
- `release()`：将资源数 +1

### 信号量为1——互斥量

当设值信号量为1，就是一个互斥量，和wait notify没有区别

```java
// 实现加一操作
public class SemaphoreX {
    // 当设值为1，代表这是一个互斥信号量
    static final Semaphore semaphore = new Semaphore(1);
    static int count = 0;
    static void addOne(){
        try {
            semaphore.acquire();
            // acquire会将信号量的计数器-1
            count ++;
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            semaphore.release();
            // release会将信号量的计数器+1
        }
    }
}
```

### 信号量实现一个对象池

**对象池**，类似于字符串常量池、线程池等等（也可以叫**限流器**）

使用池化的思想，先把对象创建出来，然后使用List保存，具体代码如下	

```java
// 使用信号量实现一个对象池
public class ObjPool<T, R> {
    // T 代表参数 R 代表返回值
    final List<T> pool;
    final Semaphore semaphore;

    public ObjPool(int size, T t) {
        this.pool = new Vector<>(size);
        // 这里用了线程安全类Vector
        // 不能使用ArrayList，因为信号量允许多个线程进入临界区，可能会导致并发操作List导致错误
        for (int i = 0; i < size; i++) {
            pool.add(t);
        }
        this.semaphore = new Semaphore(size);
    }

    R exec(Function<T,R> func){
        // Function 接口用来根据一个类型的数据得到另一个类型的数据，
        // 前者称为前置条件，后者称为后置条件
        // 类似于 R apply(T t)
        T t = null;
        try {
            semaphore.acquire();// 获取资源
            t = pool.remove(0); // 永远从队列头取
            return func.apply(t);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            pool.add(t);// 使用完要把资源还回来
            semaphore.release();
        }
        return null;
    }

    public static void main(String[] args) {
        ObjPool<Long, String> objPool = new ObjPool<>(10 ,2L);
        objPool.exec(t->{
            System.out.println(t);
            return t.toString();
        });
    }
}
```

## ReadWriteLock

> 读写锁：遵从四个个原则
>
> 1. 允许**多个线程同时读**共享变量
> 2. 只允许**一个线程写**共享变量
> 3. **写操作正在执行，那么不能读**
> 4. **读操作正在执行，那么不能写**（悲观读）

使用到了实现了`ReadWriteLock`接口的`ReentrantReadWriteLock`：

`ReadWriteLock`的API有：

- `readLock()`获取读锁
- `writeLock()`获取写锁
- `lock()`上锁
- `unlock()`释放锁
- `tryLock()`：非阻塞的获取锁
- `lockInterruptibely()`：如果线程正在等待获取锁，那么这个线程可以响应中断（别的线程可以使用`interrupt()`中断其操作）
  - `newCondition()`：**只有写锁支持生成条件**

注意：

`tryLock()`和`lock()`的区别在于：

- `tryLock()`只是"试图"获取锁, 如果锁不可用, **不会导致当前线程等待, 当前线程仍然继续往下执行代码.** （不会阻塞）
- `lock()`方法则是一定要获取到锁, 如果锁不可用, 就一直等待, 在未获得锁之前,当前线程并不继续向下执行.（阻塞）

### 读写锁实现缓存

下面的实现是一个按需加载的缓存，使用到了`ReadLock`与`WriteLock`

```java
public class MyCache<K, V> {
    final Map<K , V> cache = new HashMap<>();
    final ReadWriteLock rwl = new ReentrantReadWriteLock();
    final Lock rLock = rwl.readLock();
    final Lock wLock = rwl.writeLock();

    // 按需加载Cache
    V get(K key){
        V value = null;
        rLock.lock();
        try {
            value = cache.get(key);
        }finally {
            rLock.unlock();
        }
        if(value != null){
            // 说明缓存内存在直接返回
            return value;
        }
        // 说明缓存内部没，要去数据库读
        wLock.lock();
        try {
            value = cache.get(key);
            // 再次检查，防止别的线程也修改数据库，进行没必要的修改
            if(value == null){
                // 省略去数据库查询的代码
                value = getFromDataBase(key);
                cache.put(key, value);
            }
        }finally {
            wLock.unlock();
        }
        return value;
    }

    private V getFromDataBase(K key) {
        // 省略去数据库查询的代码
        return null;
    }

    void put(K key, V value){
        wLock.lock();
        try {
            cache.put(key, value);
        }finally {
            wLock.unlock();
        }
    }
}
```

### 读写锁的升级与降级

> 升级：就是指，在已经获取到读锁的情况下，继续获取写锁
>
> 降级：就是指，在已经获取到写锁的情况下，或许读锁

`ReentrantReadWriteLock`**只支持锁的降级，不支持锁的升级**

意思是，在已经获取到读锁后，获取写锁，是不可以的！会导致写锁永久等待，而且相关线程都会被阻塞

## StampedLock

JDK1.8提出的新锁，提供了三种模式：**写锁、悲观读锁、乐观读锁**

在`ReentrantReadWriteLock`中，提供的读锁，是悲观读的，即在读的过程中，不允许写操作

而`StampedLock`支持乐观读操作，乐观读就是认为自己读的时候不会发生写的锁，其实就是没有上锁的状态

**核心API：**

- `writeLock() readLock()`：获取写锁、读锁（如果加了try代表非阻塞的尝试获取锁），均会返回一个 **stamp**（邮戳）
- `tryOptimisticRead()`：获取乐观读锁，返回stamp；如果当前有写锁占用，那么会返回0
- `validate(long stamp)`：需要传入stamp，如果当前没有写锁占用，会返回true
- `tryConvertToWriteLock(long stamp)`：尝试**锁升级**
  - 如果当前为写锁，返回它的`stamp`
  - 如果当前为悲观读，写锁可用，那么释放读锁，返回写锁的`stamp`
  - 如果当前为乐观读，仅仅只有写锁当前立即可用的时候，才会返回写锁的`stamp`
  - 其他情况`stamp`全部返回 0

### StampedLock的官方例子

```java
class Point {
    private double x, y;
    private final StampedLock sl = new StampedLock();
    // 一个StampedLock

    // 一个点进行移动
    // 1、写锁的案例：
    void move(double deltaX, double deltaY) {
        // 写锁是一个排他锁
        long stamp = sl.writeLock();
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            sl.unlockWrite(stamp);
        }
    }
    // 计算与原点的距离
    // 2、乐观读的案例
    double distanceFromOrigin() {
        // 获取了一个乐观锁
        long stamp = sl.tryOptimisticRead();
        double currentX = x, currentY = y;
        // 把当前的位置复制一份，防止其他线程修改
        if (!sl.validate(stamp)) {
            // 如果当前锁没有被写，那么validate返回为true
            // 如果进入这个循环，说明point值已经被修改了，所以要重新获得值
            stamp = sl.readLock();// 加悲观读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                sl.unlockRead(stamp);
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
    // 尝试锁升级的案例
    void moveIfAtOrigin(double newX, double newY) { // upgrade
        // Could instead start with optimistic, not read mode

        long stamp = sl.readLock();
        try {
            while (x == 0.0 && y == 0.0) {
                long ws = sl.tryConvertToWriteLock(stamp);
                // 进行锁升级
                if (ws != 0L) {
                    // 不为0，代表锁升级成功
                    stamp = ws;
                    // 如果锁升级成功，要把新的邮戳给了stamp变量，以便后续释放
                    x = newX;
                    y = newY;
                    break;
                }
                else {
                    // 如果没有升级成功，说明当前被写锁占用
                    sl.unlockRead(stamp);
                    // 释放悲观读锁
                    stamp = sl.writeLock();
                    // 尝试获取写锁，进入等待
                }
            }
        } finally {
            sl.unlock(stamp);
        }
    }
}
```

### StampedLock的读写模板

读模板：乐观锁的实现机制，其实就是通过`stamp`，如果当前被其他线程修改了，`stamp`的值会变（类似于ABA问题的解决）

```java
final StampedLock sl = new StampedLock();
// 乐观读
long stamp = sl.tryOptimisticRead();
// 读入方法局部变量...
// 校验 stamp
if (!sl.validate(stamp)){
    // 如果当前有写锁修改
    // 升级为悲观读锁
    stamp = sl.readLock();
    try {
        // 读入方法局部变量
        .....
    } finally {
        // 释放悲观读锁
        sl.unlockRead(stamp);
    }
}
// 使用方法局部变量执行业务操作
```

写模板：

```java
long stamp = sl.writeLock();
try {
    // 写共享变量
    ......
} finally {
    sl.unlockWrite(stamp);
}
```

### StampedLock对比ReentrantReadWriteLock

`StampedLock`对比`ReentrantReadWriteLock`有了如下几点的提升：

- 支持了乐观读
- 支持锁升级

但是`StampedLock`并不能完全替代`ReentrantLock`，因为还有以下缺点：

- 不支持`Condition`
- 不是可重入锁
- 使用 `StampedLock` 一定不要调用中断操作，如果需要支持中断功能，一定使用可中断的悲观读锁 `readLockInterruptibly() `和写锁 `writeLockInterruptibly()`

| 对比项          | `StampedLock`            | `ReentrantReadWriteLock` |
| --------------- | ------------------------ | ------------------------ |
| 模式            | 三种：写、悲观读、乐观读 | 两种：写、悲观读         |
| 支持`Condition` | 不支持                   | 只有写锁支持生成         |
| 是否可重入      | 不可重入                 | 可重入                   |
| 锁升级          | 支持                     | 不支持                   |
| 锁降级          | 支持                     | 支持                     |

## CountDownLatch

可以实现让一个线程等待其他线程完成后再执行

假设我们要实现一个**对账系统**

![对账逻辑](http://img.yesmylord.cn//image-20210906194144240.png)

使用`CountDownLatch`我们可以很好的实现这个案例

核心API：

- `new CountDownLatch(int count)`：构造一个要等待几个任务的CountDownLatch
- `countDown()`：将count值 -1
- `await()`：进入阻塞状态，直到count值变为0，才会允许通过

```java
Order order; // 模拟订单类
Bill bill; // 模拟账单类
public void check() throws InterruptedException {
    Executor executor = Executors.newFixedThreadPool(2);
    while(hasBill()){
        // 计数器初始化为 2
        CountDownLatch latch = new CountDownLatch(2);
        // 查询未对账订单
        executor.execute(()-> {
            bill = getUnCheckedBill();
            latch.countDown();
        });
        // 查询派送单
        executor.execute(()-> {
            order = getOrder();
            latch.countDown();
        });
        // 等待两个查询操作结束
        latch.await();
        // 执行对账操作
        Diff diff = check(order, bill);
        // 差异写入差异库
        save(diff);
    }
}
```

> 如果不用CountDownLatch我们怎么实现这个案例？

可以使用两个线程分别执行查订单，查账单的事情，然后调用`join()`方法，让主线程等待两个线程完成后再继续执行

## CyclicBarrier

类似于CountDownLatch，为了解决其不能重复使用的问题而提出的

构造方法特别重要，两个参数：第一个就是等待的值，第二个是希望完成后执行的内容（是一个Runnable接口）

有两个核心API：

- `await()`：执行完成自动将值减去1
- `reset()`：不用我们自己调用

### 任务加载Demo

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
                //cyclicBarrier.reset();// 重置屏障，不需要我们自己调用，await会自己-1
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

## FutureTask

之前说到过，实现`Callable`接口，实现`call`方法调用，就可以实现一个有返回值的线程，这个返回值就是一个`Future`接口对象

---

先来介绍`Future`接口：`Future`接口有五个方法

- `get()`：获取值，如果获取时，线程还没有执行完成，那么会进入阻塞状态
- `get(timeout, timeunit)`：设置阻塞的超时时间
- `cancel()`：可以取消任务执行
- `isCanceled()`：判断任务是否取消
- `isDone()`：判断任务是否执行结束

而`FutureTask`就是一个工具类，实现了`Future`接口，使用看Demo吧

### 泡茶Demo

![分工图](http://img.yesmylord.cn//image-20210911164234074.png)

最优烧开水程序：

```java
// 实现烧开水程序
public class No09FutureTask {
    static FutureTask<String> ft1 = new FutureTask<String>(new T1Task());
    static FutureTask<String> ft2 = new FutureTask<String>(new T2Task());
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Thread t1 = new Thread(ft1);
        Thread t2 = new Thread(ft2);
        t1.start();
        t2.start();
        System.out.println(ft1.get());
    }
    static class T1Task implements Callable<String>{

        @Override
        public String call() throws Exception {
            System.out.println("T1：洗水壶");
            Thread.sleep(1000);
            System.out.println("T1：烧开水");
            Thread.sleep(15000);
            String tea = ft2.get();
            System.out.println("T1：拿到茶叶"+ tea);
            return "上茶"+tea;
        }
    }
    static class T2Task implements Callable<String>{

        @Override
        public String call() throws Exception {
            System.out.println("T2：洗茶壶");
            Thread.sleep(1000);
            System.out.println("T2：洗茶杯");
            Thread.sleep(2000);
            System.out.println("T2：拿茶叶");
            Thread.sleep(1000);
            return "普洱";
        }
    }
}
```











