---
title: Java进阶
date: 2021-08-04 22:03:51
tags: 
- Java
categories: 
- 后台
- Java

---

<center>
引言:Java进阶，从问题来复习基础
</center>
<!--more-->

# Java基础

### Java是解释执行吗？

答：不完全是

- 老版Java解释执行我们写的java代码，在Javac编译后，被JVM解释执行
- Hotspot虚拟机实现JIT，对于热点代码，它会在运行时将其编译为机器码，直接跑在机器上
- JDK9 又有了一种新型的编译方式**AOT**（Ahead of  Time Compilation），直接将字节码转化为机器码

### Exception与Error

- 都是`Throwable`类的子类，只有继承了`Throwable`才能被抛出或者捕获
- `Error`指正常情况下难以预料的异常，交由父类或JVM来帮我们处理
- `Exception`指可以预见的异常，应该我们自己处理。
- `Exception`分为**检查型异常**和**非检查型异常**（或者说编译时异常、运行时异常）
  - 检查型异常必须**显式的捕获**处理（编译时异常）
  - 非检查型异常就是运行时异常，通常是编码可以避免的逻辑错误（运行时异常）
- 常见的错误：
  - **Error**：OOM、SOF、NoClassDefFoundError、ExceptionInInitializerError（静态变量初始化的过程中出现了异常）
  - **Exception**：
    - **检查型异常**：IOException、ConcurrentModificationException 
    - **运行时异常**（非检查型异常）：NullPointException、ClassCastException
- `NoClassDefFoundError`与`NoClassFoundExeption`：区别
  - `NoClassDefFoundError`：运行时JVM加载不到类或者找不到类
  - `NoClassFoundExeption`：编译时JVM加载不到类或者找不到类导致的

- 捕获异常要注意
  - 不要捕获大的异常比如Exception
  - 不要生吞异常（连记录都不记录）

### final、finall、finalize

搞不懂为什么要将这三个进行分析。。。

### 强、软、弱、虚引用

​		虚引用补充：比如替代finalize方法的Cleaner，就用到了虚引用，就是用来通知此类，如果这个类被回收了，我能收到消息

`-XX:+PrintReferenceGC`可以查看引用信息

### String

- `intern`：是一种**显式的排重机制**（因为intern会将字符串放进池，这样不会导致有重复的String）
- String变量



### JDK动态代理与cglib

- JDK动态代理，只能代理实现了接口的类
- cglib代理的类可以不实现接口

### 包装类

- Integer：-128 到 127；缓存范围可以调整`-XX:AutoBoxCacheMax=N`
- Boolean缓存两个实例 True和False
- booleanJVM当做int类型处理，占4个字节，但是如果是一个boolean数组，每个就只占1字节的大小，JVM会对其进行调节
- 装箱：`Integer.valueOf`；拆箱：`Integer.intValue`

- 自动装箱拆箱发生在什么阶段？

### Vector、ArrayList、LinkedList

- Vector扩容时扩容1倍
- ArrayList扩容扩1.5倍
- 集合类下有：list、queue、set（注意：map不是集合包下的）
- LinkedList用双向链表实现

### Java IO

- 传统IO：FIle、Reader/Writer、Output/InputStream、BufferedOutputStream
- 1.4 引入 NIO：Buffer、Channel、Selector（实现**多路复用**的基础）、Charset
- NIO同步非阻塞IO

![image-20210824113943541](http://img.yesmylord.cn//image-20210824113943541.png)

