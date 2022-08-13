---
title: Netty
date: 2021-10-06 13:42:36
tags: 
- Java
- Netty
categories: 
- Java
- Netty

---

<center>
引言：Netty
</center>
<!--more-->

# Netty

## Netty概述

> Netty是什么？
>
> 一个**异步**的、基于**事件**驱动的网络应用框架

- **异步**：通过回调+`Future`实现
- **事件**：通过`ChannelHandler`实现

（Netty在网络应用框架的地位等同于Spring在JavaEE的地位）

> Netty如何引入？

Maven引入包，即可使用Netty

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
```

> Netty的构成

Netty主要由五大部分组件构成：（下面我们会依次介绍）

1. EventLoop & EventLoopGroup
2. Channel & ChannelFuture
3. Future & Promise
4. Handler & PipeLine
5. ByteBuf

## EventLoop & EventLoopGroup

> EventLoop（事件循环）：相当于一个**单线程执行器**，维护了一个Selector
>
> （我们可以把EventLoop当做一个`Thread+Selector`）

作用：内部有`run`方法**处理`channel`上源源不断的IO事件**

### 基本API

EventLoop的继承：

- 继承`j.u.c.ScheduledExecutorService`，有关于定时线程的相关方法
- 继承`EventExecutor`，内部有方法可以查看EventLoop属于的组，以及判断一个线程属不属于此`EventLoop`

核心API：

- 其可以通过`EventLoop`的`next`方法获得
- （因为继承了ScheduledExecutorService，所以有执行任务的方法）
- `excute(Runnable)`执行任务
- `submit(Runnable)`执行任务，返回一个Future对象
- `scheduleAtFixedRate(Runnable, initialDelay, period, TimeUnit)`执行定时任务

> EventLoopGroup（事件循环组）：内部包含一组EventLoop

这个接口的实现类有很多，常用的实现类有：

- `NioEventLoopGroup`：负责处理**IO事件、普通任务、定时任务**
- `DefaultEventLoopGroup`：负责处理**普通任务、定时任务**

核心API：

- `new NioEventLoopGroup(int)`：创建一个NioEventLoopGroup，参数是int类型的值，如果不传，按默认的值来设置

  ```java
  DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt("io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));
  // 默认是1或是配置文件配置的线程数或是电脑核心线程数*2的较大值
  ```

- `next()`：可以获得一个**下一个**EventLoop对象

---

这里简单的一个Demo介绍一下如何使用：	

```java
EventLoopGroup group1 = new NioEventLoopGroup(5); //此对象可以处理： io事件、普通任务、定时任务
EventLoopGroup group2 = new DefaultEventLoopGroup(); //此对象可以处理： 普通任务、定时任务
group1.execute(() -> {
    System.out.println("执行普通任务");
});
group1.next().scheduleAtFixedRate(() -> {
    System.out.println("执行定时任务");
}, 0, 1, TimeUnit.SECONDS);
```

`NioEventLoopGroup`最主要的作用是处理IO事件，下面我们会单独专门讲述

### 执行IO请求

NioEventLoopGroup可以处理三种任务：IO事件、普通任务、定时任务

处理IO事件不是`NioEventLoopGroup`一个就可以完成的，他需要与其他组件合作

```java
new ServerBootstrap()
    .group(new NioEventLoopGroup())
    // 这一步就是创建EventLoopGroup
    .channel(NioServerSocketChannel.class)
    // 设置通道
    .childHandler(new ChannelInitializer<NioSocketChannel>() {
        @Override
        protected void initChannel(NioSocketChannel ch) throws Exception {
            // 进行处理
            ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                    ByteBuf byteBuf = (ByteBuf) msg;
                    log.debug(byteBuf.toString(Charset.defaultCharset()));
                }
            });
        }
    })
    .bind(8080);
```

### 任务分工与细化

#### 分工

NIO一节，我们学过`boss`与`worker`模型，boss负责ACCEPT请求，worker负责读写请求，在Netty的任务可以细分：

```java
new ServerBootstrap()
    // 这里可以指定两个EventLoopGroup
    // 第一个就是boss，第二个就是worker
    .group(new NioEventLoopGroup(), new NioEventLoopGroup())
```

> 如果第一个参数是boss，只负责ACCEPT请求，那么我们是不是要设置参数为1呢？比如这样`.group(new NioEventLoopGroup(1), new NioEventLoopGroup())`

不需要的，Netty默认就是Boss只会被启动一个，所以可以不用指定1

#### 细化

> 设想这么一个场景，如果有比较重量级的操作，我们可以单独给其分配一个EventLoopGroup来专门执行这种重量级操作，以免阻塞我们的其他任务

```java
//【细化】让一个group专门去做耗时的工作
EventLoopGroup group = new DefaultEventLoop();
new ServerBootstrap()
    // 【分工】：这里可以指定两个EventLoopGroup
    // 第一个就是boss，第二个就是worker
    .group(new NioEventLoopGroup(), new NioEventLoopGroup())
    .channel(NioServerSocketChannel.class)
    .childHandler(new ChannelInitializer<NioSocketChannel>() {
        @Override
        protected void initChannel(NioSocketChannel ch) throws Exception {
            ch.pipeline().addLast("handler1",new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                    ByteBuf byteBuf = (ByteBuf) msg;
                    log.debug(byteBuf.toString(Charset.defaultCharset()));
                    System.out.println("handler1");
                    ctx.fireChannelRead(msg);// 这个方法可以传msg到下一个handler
                }
            }).addLast(group,"handler2", new ChannelInboundHandlerAdapter(){
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                    System.out.println("handler2");
                    System.out.println(msg);
                }
            });
        }
    })
    .bind(8080);
```

## Channel & ChannelFuture

> channel： **一个到实体的开放连接**
>
> 实体包括一个硬件设备、一个文件、一个Socket等等内容

类似于NIO的通道，Netty的Channel本身也是对Channel的一个封装

### 核心API

- `close()`可以用来关闭channel
- `closeFuture()`：用来处理channel的关闭，可以附加其他操作
  - `sync`方法作用是**同步**等待channel关闭
  - `addListener`**异步**等待channel关闭
- `pipeline()`：方法**添加处理器**
- `write()`方法将数据写入（**写入发送缓冲区**，但不一定立即发送，可能达到一定大小，才会发送出去）
- `writeAndFlush()`将数据立刻写入并刷出（写入缓冲区并且立即发送）
  - 这个方法相当于调用`write()`与`flush()`两个方法

### ChannelFuture对象

> ChannelFuture：异步IO操作的返回结果（成功、失败、或是取消）

​		由于Netty中所有的IO操作都是异步的，这意味着任何IO调用都将立即返回，但不保证请求的IO操作已在调用结束时完成

​		**所以就有了`ChannelFuture`这个对象，用于在某个时间点确定操作结果**

理解Netty的操作都是异步的很重要，比如我们的客户端连接操作

```java
ChannelFuture channelFuture = new Bootstrap()
    .group(new NioEventLoopGroup())
    .channel(NioSocketChannel.class)
    .handler(new ChannelInitializer<NioSocketChannel>() {
        @Override
        protected void initChannel(NioSocketChannel ch) throws Exception {
            ch.pipeline().addLast(new StringEncoder());
        }
    })
    .connect(new InetSocketAddress("localhost", 8080));
// 获得ChannelFuture对象
channelFuture
    //.sync() // 我们现在不去调用同步方法
    .channel()
    .writeAndFlush("hello world");
```

执行会发现，不会给服务器发送hello world的信息

> 为什么不会给客户端不会发送hello world的信息？

​		因为Netty是异步非阻塞的，`main`线程发起调用，但其实是创建了一个新的NIO线程来执行`connect()`操作（异步），这个连接操作很耗时

​		而`main`线程调用完后，会立即向下执行（非阻塞），因此获得的channel对象不是成功建立连接后的对象，它发消息也就发不出去了

> 如何解决这个问题？

- 【法一】调用`sync`方法，这个方法会让main线程与运行connect的线程同步（即阻塞`main`线程直到`channel`创建完成）
- 【法二】调用`addListener(回调对象)`，传入一个`GenericFutureListener`接口，我们可以传入其子接口`ChannelFutureListener`

```java
channelFuture.addListener(new ChannelFutureListener() {
    @Override
    // operationComplete方法会由执行NIO的线程执行完成后调用
    public void operationComplete(ChannelFuture future) throws Exception {
        Channel channel = future.channel();
        channel.writeAndFlush("hello");
    }
});
```

> 注意：
>
> ​		`Channel`对象的关闭与连接一样，都是由另外的线程真正执行的关闭操作，`Channel`对象可以调用`sync`同步或者`addListener`异步来执行通道关闭后的操作

## Future & Promise

> `Future`是另一种在操作完成时通知APP的方式
>
> （还有一种是回调方法,比如新的连接建立触发：`channelActive()`）

### 两个Future与Promise的关系

JUC也有Future对象，Netty的Future继承了JUC的Future，Promise是对Netty Future的进一步扩展

> 一句话：`Promise`继承 netty`Future`；netty `Future` 继承 JUC `Future`

区别：

- JDK `Future`只能同步等待任务结束，才能得到结果
- Netty `Future`可以同步/异步等待任务结束，然后获得结果
- `Promise`不仅有Future的功能，而且**脱离了任务独立存在，作为两个线程间传递结果的容器**

核心API对比：

| 功能          | JDK Future                    | Netty Future                                 | Promise            |
| ------------- | ----------------------------- | -------------------------------------------- | ------------------ |
| `cancel`      | 取消任务                      | 继承                                         | 继承               |
| `isCanceled`  | 判断任务是否取消              | 继承                                         | 继承               |
| `isDone`      | 判断任务是否结束（成功/失败） | 继承                                         | 继承               |
| `get`         | 获得结果（阻塞等待）          | 继承                                         | 继承               |
| `getNow`      | 无                            | 获取任务结果，非阻塞，会先立即返回null       | 继承               |
| `await`       | 无                            | 等待任务结束（任务失败**不会抛出异常**）     | 继承               |
| `sync`        | 无                            | 等待任务结束（任务失败**抛出异常**）         | 继承               |
| `isSuccess`   | 无                            | 判断任务是否成功                             | 继承               |
| `cause`       | 无                            | 获取失败信息（非阻塞），如果没有失败返回null | 继承               |
| `addListener` | 无                            | 添加回调，异步接收结果                       | 继承               |
| `setSuccess`  | 无                            | 无                                           | 设置成功返回的结果 |
| `setFailure`  | 无                            | 无                                           | 设置失败返回的结果 |

## 事件 & ChannelHandler & ChannelPipeLine

> 事件：事件是某些动作完成时发送的信息

入站端的事件比如有：连接已激活或失活、数据读取、用户事件、错误事件

出站事件：打开关闭远程节点的连接、数据写到或冲刷到套接字

> `ChannelHandler`：处理IO事件或拦截IO操作，并将其转发到其PipeLine中的下一个handler（可以理解为一道工序）

简单的我们可以理解为：是**一个可以响应不同事件的回调方法**

这里有个图，很好的表示了事件和`ChannelHandler`的关系

![事件与ChannelHandler](http://img.yesmylord.cn//image-20220801201224254.png)

如图所示，在一个事件发生后，会经过一系列的处理器（即Handler）最后成功入站或出站

> **注意**：Netty服务器至少需要两部分：
>
> - 至少一个 `ChannelHandler`
> - 引导：客户端Bootstrap、服务端ServerBootstrap
>
> 引导的作用：为了配置服务器，比如连接服务器到指定的端口

> `ChannelPipeLine`：一个Handler的列表（可以理解为一条流水线）或者说是ChannelHander的容器

### InboundHandler

有两种`ChannelHandler`，分为入站、出站两种（注意是站，不是栈）

#### ChannelInboundHandlerAdapter

> `ChannelHandler`是一个父接口，使用需要重写很多方法，所以实现时，我们可以使用他们的适配器`Adapter`的子类，来很快的达到目的

入站处理器通常是`ChannelInboundHandlerAdapter`的子类，下面是它的相关事件的**回调**

（所谓回调：当触发对应事件的时候，会调用相关方法）

- `channelRead()`：每个消息传入都会调用此方法
- `channelReadComplete()`：当前批量读取的最后一条消息会触发此方法
- `exceptionCaught()`：读取操作发生异常会调用此方法

#### SimpleChannelInboundHandler

此类是`ChannelInboundHandlerAdapter`的子类，它的相关回调有：

- `channelActive()`：当连接建立后将被调用
- `channelRead0()`：收到一条消息就被调用
- `exceptionCaught()`：读取操作发生异常会调用此方法

> 注意：`channelRead0`可能会执行很多次，因为由服务器发送的消息可能会被分块接收。
>
> 也就是说，如果服务器发送了 5 字节，那么不能保证这5字节会被一次性接收

那么`SimpleChannelInboundHandler`和他的父类`ChannelInboundHandlerAdapter`有什么区别呢？

前者会帮我们释放关于`ByteBuf`的内存引用，而后者没有实现。



### OutboundHandler

出站处理器通常是`ChannelOutboundHandlerAdapter`的子类，主要用来对写回数据进行加工

通过通道获取PipeLine，通过PipeLine的`addLast`方法添加处理器

### ChannelPipeline与Handler的执行流程

PipeLine（或者说Handler的执行流程）的结构就是一个**双向链表**

两个特殊的Handler：一个`head`，一个`tail`

![PipeLine结构](http://img.yesmylord.cn//image-20211008204203210.png)

注意：

- `pipeline.addLast(xxx)`：会将handler加到tail之前，并不是真正的最后
- `Inbound`负责执行**读操作**，是从`head`开始的
- `OutBound`负责执行**写操作**，是从`tail`开始的

> `Inboundhandler`内一个`handler`如何传递对象给下一个`handler`？

可以通过调用`super.channelRead(ctx,msg);`方法，将想要传递的对象作为`msg`传递给下一个`handler`，

其**内部其实调用的是`fireChannelRead`方法**

```java
@Skip
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    ctx.fireChannelRead(msg);
}
```

如果某一个`handler`没有调用此方法，那么传递链就会断开

> `channel`的`write`与`ctx`的`write`的区别！（重要）

这两个容易搞混，如图示：

- `channel`的`write`是从tail开始向前找到第一个`Outboundhandler`（注意：是从tail开始！找的`OutBoundHandler`，不找`InboundHandler`）
- `ctx`的`write`是从当前`handler`开始向前找`Outboundhandler`（注意：是从当前`handler`开始，也是找的`outboundhandler`）

## ByteBuf

ByteBuf是netty对NIO ByteBuffer的一个增强

### ByteBuf的创建

创建一个`ByteBuf`不能直接new，需要`ByteBufAllocator`来创建

- 使用`ByteBufAllocator.DEFAULT.buffer()`
- 默认字节数组长度256字节，可以自己指定
- 如果存满，会自动扩容（NIO会报错）

```JAVA
// 可以传入一个Capacity，默认是256字节，而且可以扩容
ByteBuf byteBuf = ByteBufAllocator.DEFAULT.buffer();
System.out.println(byteBuf);
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 300; i++) {
    sb.append("2");
}
byteBuf.writeBytes(sb.toString().getBytes());
System.out.println(byteBuf);
/* 输出：（ridx读指针、widx写指针）
        PooledUnsafeDirectByteBuf(ridx: 0, widx: 0, cap: 256)
        PooledUnsafeDirectByteBuf(ridx: 0, widx: 300, cap: 512)
*/
```

- 默认创建的就是**直接内存**，也可以创建**堆内存**

```java
ByteBufAllocator.DEFAULT.buffer();// 默认就创建直接内存
ByteBufAllocator.DEFAULT.directBuffer();// 创建直接内存
ByteBufAllocator.DEFAULT.heapBuffer();// 创建堆内存
```

> 直接内存与堆内存的区别：

1、管理机制不同：直接内存由OS管理，堆内存由JVM管理

2、创建速度：直接内存创建比较麻烦，堆内存创建十分迅速

综上：

直接内存的优点是读写性能好（会少一次数据Copy的过程），但是创建比较慢，而且需要自己进行释放

而堆内存的优点是创建速度快，但是读写性能会低一点

---

PS：此处再贴一个调试ByteBuf的小工具

```java
import static io.netty.buffer.ByteBufUtil.appendPrettyHexDump;
import static io.netty.util.internal.StringUtil.NEWLINE;

private static void log(ByteBuf buffer){
    int length = buffer.readableBytes();
    int rows = length / 16 + (length % 15 == 0 ? 0 : 1) + 4;
    StringBuilder buf = new StringBuilder(rows * 80 * 2)
        .append("read index").append(buffer.readerIndex())
        .append(" write index").append(buffer.writerIndex())
        .append(" capacity").append(buffer.capacity())
        .append(NEWLINE);
    appendPrettyHexDump(buf, buffer);
    System.out.println(buf.toString());
}
```

### 池化思想

> 为什么ByteBuf引入池化思想？

Netty采用了直接内存，但是直接内存的缺点是创建比较慢，所以引入了池化思想，来优化

池化功能在4.1之前不成熟，默认是非池化的

**在4.1之后，默认是池化创建（非Android平台）**

可以配置参数设置是否开启池化

### ByteBuf的组成

ByteBuf由四部分组成：废弃部分、可读部分、可写部分、可扩容部分

![ByteBuf的组成](http://img.yesmylord.cn//image-20211008213401717.png)

- 有读指针和写指针

- 最大容量代表int类型的最大值（我们的内存可能都没有那么大）

> 相比较NIO 的Buffer，ByteBuf有什么优势呢？

1. 可以扩容（有可扩容部分）
2. 不必再来回切换读写模式（有读指针和写指针）

> ByteBuf的扩容规则

写入后容量大小是否超过512？

- 未超过：则选择下一个16的整数倍

- 超过：选择下一个`2^n`

注意：扩容不能超过最大容量，否则报错

​		比如，假设当前为默认容量256，在写入一个数据后，为257，此时触发扩容，由于小于512，所以会选择下一个16的倍数，即272

​		一直写入，写入到513字节时，触发扩容，扩容为2的幂次，即1024。



### 核心API

> 写操作

ByteBuf给每一个基本数据类型都提供了写方法，此处捡重点说一下

- `writeBoolean(boolean)`：ByteBuf内部存储时，使用0表示false，1表示true
- `writeInt(int) & writeIntLE(int)`：这两个方法有什么区别呢？
  - `writeInt(int)`就是常用的方法，他是**大端存储**的
  - `writeIntLE(int)`是小端存储的

大端存储6，其结构就是`0000 0110`，而小端存储为`0110 0000`

（一般内存设计都是大端存储的，极个别的内存厂商使用小端存储）

- `writeBytes(xxx)`：这个方法可传入的参数非常多，甚至包括NIO的ByteBuf
- `writeCharSequence(CharSequence子类, 字符集)`：这个方法可以传入字符串等等CharSequence的子类

> 读操作

读操作主要的方法有三个

- `readByte()`：读取1个字节
- `readInt()`：读取4个字节
- `markReaderIndex()`：标记当前的读位置
- `resetReaderIndex()`：重置读位置到标记的位置

### ByteBuf的销毁与内存释放

ByteBuf的种类：

- `UnpooledHeapByteBuf`：使用JVM内存，只需等待GC
- `UnpooledDirectByteBuf` 使用直接内存，也可以等待GC，但是这样回收不及时，需要特殊的方法来回收内存
- `PooledByteBuf` 和它的子类使用了池化机制，需要更复杂的规则来回收内存

> `Netty`采用了**引用计数法**来控制回收内存

**每个`ByteBuf`都实现了`ReferenceCounted`接口**

- 每个`ByteBuf`对象初始计数为1
- 调用`release`方法将计数减1，如果计数为0，`ByteBuf`内存被回收
- 调用`retain`方法计数加1，表示调用者没用完之前，其他`handler`即使调用了`release`也不会造成回收

- 如果计数为0，那么它的内存就会被回收，即使ByteBuf对象还在，它的方法也不能正常使用

```java
public interface ReferenceCounted {
    boolean release();
    ReferenceCounted retain();
    ...
}
```

> handler释放ByteBuf的规则是什么？

因为pipeLine上有很多handler，所以我们要保证所有的handler都不会再使用ByteBuf了，再去释放

因此规则就是：**谁最后使用，谁释放**

> `head` handler与`tail`handler也会帮我们进行对ByteBuf的处理

`tail`handler：

学习handler一节我们知道，channel的write方法，会先找到`tail`，所以tail其实也是一个`Inboundhandler`，因此我们主要关注`channelRead()`方法

```java
// A special catch-all handler that handles both bytes and messages.
final class TailContext extends AbstractChannelHandlerContext implements ChannelInboundHandler {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        onUnhandledInboundMessage(ctx, msg);
    }
    ...省略其他方法...
}   
```

到最后会遍历到一个方法`ReferenceCountUtil.release()`，此方法如下

```java
public static boolean release(Object msg) {
    if (msg instanceof ReferenceCounted) {
        // 如果msg是ByteBuf的实例，那么就将其release，将计数-1
        // 每一个ByteBuf都实现了ReferenceCounted
        return ((ReferenceCounted) msg).release();
    }
    return false;
}
```



`head` handler：

`head`handler既是一个`InboundHandler`，也是一个`OutboundHandler`

```java
final class HeadContext extends AbstractChannelHandlerContext
    implements ChannelOutboundHandler, ChannelInboundHandler {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
        unsafe.write(msg, promise);
    }
}
```

找到这个方法的实现类（这个方法实现涉及到了`outboundBuffer`，我们之后再了解）

```java
public final void write(Object msg, ChannelPromise promise) {
    assertEventLoop();

    ChannelOutboundBuffer outboundBuffer = this.outboundBuffer;
    if (outboundBuffer == null) {
        // If the outboundBuffer is null we know the channel was closed and so
        // need to fail the future right away. If it is not null the handling of the rest
        // will be done in flush0()
        // See https://github.com/netty/netty/issues/2362
        safeSetFailure(promise, newClosedChannelException(initialCloseCause));
        // release message now to prevent resource-leak
        ReferenceCountUtil.release(msg);
        // 此处进行了删除
        return;
    }

    int size;
    try {
        msg = filterOutboundMessage(msg);
        size = pipeline.estimatorHandle().size(msg);
        if (size < 0) {
            size = 0;
        }
    } catch (Throwable t) {
        safeSetFailure(promise, t);
        ReferenceCountUtil.release(msg);
        return;
    }

    outboundBuffer.addMessage(msg, size, promise);
}
```

### ByteBuf的零拷贝

Netty的`ByteBuf`的零拷贝体现，与NIO的有所不同

在NIO中的零拷贝，意思是不会将重复的数据拷贝到用户内存中；

在`ByteBuf`中的零拷贝，意思是**仅仅是逻辑上的复制或拷贝**（**仅仅只有读写指针是相互独立的，但是内存其实共用的一块内存**）

> 零拷贝体现一：`slice(index, length)`

获取源byteBuf的一部分，两者共用同一块内存，仅仅是读写指针独立

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer();
buffer.writeCharSequence("abcde", StandardCharsets.UTF_8);
ByteBuf slice1 = buffer.slice(2, 2);//cd
buffer.setByte(2, 'C');// 更改了源，slice也会变
```

由于ByteBuf这种零拷贝特性，如果发生这种情况就会出错：

```java
buffer.release();
log(slice1);// 源bytebuf已经释放，引用slice就会报错
```

所以为了防止这种问题，所以我们应该在`slice`后，调用一次`retain()`方法

> 零拷贝体现二：`duplicate()`

将源数据整个复制一份，其他遇`slice`一样，可以理解为`slice(0, buffer.capacity())`

> 零拷贝体现三：`CompositeByteBuf`

与`slice`正好相反，这个方法是将多个buf合为一个buf，不会发生复制操作，也是逻辑上的合并

```java
// 创建两个buf等待合并操作
ByteBuf buffer1 = ByteBufAllocator.DEFAULT.buffer();
buffer1.writeBytes(new byte[]{1,2,3,4,5});
ByteBuf buffer2 = ByteBufAllocator.DEFAULT.buffer();
buffer2.writeBytes(new byte[]{6,7,8,9,10});

ByteBuf bufUnion1 = ByteBufAllocator.DEFAULT.buffer();
bufUnion1.writeBytes(buffer1).writeBytes(buffer2);
// 这种方式也可以实现合并，但是发生了复制，影响性能
CompositeByteBuf bufUnion2 = ByteBufAllocator.DEFAULT.compositeBuffer();
bufUnion2.addComponents(true,buffer1, buffer2);
// 这种方式就是零拷贝的方式
```

`addComponents()`第一个参数为true代表自动调整写指针（默认是不会调整写指针的）

虽然提高了性能（零拷贝），但是不便于维护

---

不同与零拷贝，ByteBuf实现了`copy()`函数，进行**深拷贝**





