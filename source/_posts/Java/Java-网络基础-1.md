---
title: Java-网络基础-1
date: 2019-08-24 11:40:15
tags: 
- Java
- Java网络基础
- Socket
categories: 
- 后台
- Java
---

<center>
引言：网络编程入门
</center>
<!--more-->

# TCP通信程序

TCP能实现两台计算机之间的数据交互，严格区分客户端与服务端

**两端通信时步骤**：

1. 服务端程序，需要事先启动
2. 等待客户端的连接
3. 客户端主动连接服务器端，连接成功才能通信，服务端不能主动连接客户端
4. 这个连接中包含一个对象，这个对象就是IO对象，客户端可以使用IO对象进行通信
5. 通信的数据不仅仅是字符，所以IO对象是**字节流对象**

**服务器必须明确两件事情**:
1. 多个客户端同时和服务器进行交互，服务器必须明确是哪一个用户

    在服务器端有一个方法，叫做`accept`客服端获取到请求的客户端对象
2. 多个客户端同时和服务器交互，就需要使用到多个IO流对象

    服务器是没有IO流的，服务器可以获取到请求的客服端对象`Socket`使用每个客服端`Socket`中提供的`Socket`流
    
    服务器使用端的字节输入流读取客户端发送的数据，服务器使用客户端的字节输出流给客户端写数据
    
    即：服务器使用客户端的流与客户交流

**java中提供了两个类来实现TCP通信**:
1. 客户端：`java.net.Socket`实现客户端的**套接字**，套接字是两台机器之间通信的端点，包含了IP地址和端口号的网络单位
2. 服务器端：`java.net.ServerSocket`



## 操作步骤

### 客户端
1. 创建一个客户端对象`Socket`，构造方法绑定服务器的IP地址和端口号
2. 使用`Socket`对象中的方法`getOutputStream()`获取`OutputSteam`对象
3. 使用`OutputStream`对象中的方法`write`给服务器发送数据
4. 使用`Socket`对象中的方法`getInputStream()`获取`InputStream`对象
5. 使用`InputStream`对象中的方法`read`读取服务器回写的数据
6. 释放资源`Socket`

### 服务器端

1. 创建服务器`ServerSocket`对象和系统要指定的端口号
2. 使用`ServerSocket`对象中的方法`accept`，获取到请求的客户端对象`Socket`
3. 使用`Socket`对象中的方法`getInputStream()`获取`InputSteam`对象
4. 使用`InputStream`对象中的方法`read`读取客户端发送的数据
5. 使用`Socket`对象中的方法`getOutputStream()`获取`OutputStream`对象
6. 使用`OutputStream`对象中的方法`write`，给客户端回写数据
7. 释放资源`Socket,ServeSocket`

# Socket

## 包路径
```
java.net
```

## 构造方法
```java
public Socker()
//创建一个还没有被连接的套接字

public Socket(String host, int port)
/*
    host 是服务器主机的名称/服务器的IP地址
    port 服务器的端口号
*/
```
## 成员方法
```java
public InputStream getInputStream() throws IOException
//返回此套接字的输入流。 

public OutputStream getOutputStream() throws IOException
//返回此套接字的输出流。 

public void close() throws IOException
//关闭此套接字

public void setSoTimeOut(int timeOutMilliseconds)
//设置该套接字的一个等待时间

public void connect(SocketAddress address)
//将该套接字连接到指定地址

public void connect(SocketAddress address,int timeOutInMilliseconds)
//将该套接字连接到指定地址，若在给定的事件没有响应则抛出一个InterruptedIOExeption

public boolean isConnected()
//如果该套接字已经被连接，返回true

public boolean isClosed()
//如果套接字已经被关闭，返回true
```


注意：
1. 客户端和服务器交互，必须使用Socket，Socket中提供的网络流不能使用自己创建的流对象
2. 当我们创建客户端对象Socket的时候，就会去请求服务器和服务器经过 三次握手 建立连接通路
   
    这时如果服务器没有启动就会抛出异常

    如果服务器已经启动则可以进行交互
## 半关闭

> 半关闭(half-close)：套接字连接的一端可以终止其输出，同时仍可以接收来自另一端的数据


```java
/*
    Socket的方法
*/
public void shutdownOutput()
//将输出流设为流结束
public void shutdownInput()
//将输入流设为流结束

public boolean isOutputShutdown()
//如果输出流已被关闭，则返回true
public boolean isInputShutdown()
//如果输入流已被关闭，则返回true

```


# InetAddress
在主机名和因特网地址之间进行转换

## 包路径
```
java.net
```
## 构造方法
```java
static InetAddress getByName(String host)
//静态方法，返回一个InetAddress对象，该对象封装了一个4字节的序列

static InetAddress[] getAllByName(String host)
/*
    一些较大的主机名会对应多个因特网地址，可以使用这个方法来获得所有的主机
*/
```
## 常用方法
```java
static InetAddress getLocalHost()
/*
    有时我们需要本地地址的地址，如果只是要求得到localhost的地址，那总会得到本地回环地址127.0.0.1，但是其他程序无法用这个地址来连接到这台主机，所以我们可以使用getLocalHost方法来得到本地主机的地址
*/

byte[] getAddress()
//返回一个包含数字地址的字节数组，是byte型数组，超过-128~127这个范围的数字都会以负数表示

String getHostAddress()
//返回一个由十进制数组成的字符串，如“129.15.2.53”

String getHostName()
//返回主机名称
```
## 示例代码
```java
public class InetAddressTest {
    public static void main(String[] args) throws UnknownHostException {
        String host = "baidu.com";
        InetAddress[] allByName = InetAddress.getAllByName(host);
        for (InetAddress inetAddress : allByName) {
            System.out.println(inetAddress);
            /*
                baidu.com/39.156.69.79
                baidu.com/220.181.38.148
                主机名/地址
            */
        }

        InetAddress localHost = InetAddress.getLocalHost();
        byte[] address = localHost.getAddress();
        for (byte b : address) {
            System.out.println(b);
        }
    }
}
```


# ServerSocket
此类实现服务器套接字。服务器套接字等待请求通过网络传入。

它基于该请求执行某些操作，然后可能向请求者返回结果
## 包路径
```
java.net
```
## 构造方法
```java
public ServerSocket(int port) throws IOException
//创建一个监听端口的服务器套接字。端口 0 在所有空闲端口上创建套接字
```
## 常用方法

```java
public Socket accept()throws IOException
//侦听并接受到此套接字的连接。此方法在连接传入之前一直阻塞,当有客户端连接到该端口上，则返回一个Socket对象
```

# 实现服务器和客户端的交流
要先运行服务器，再运行客户端

服务器
```java
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @author BlackKnight
 */
public class TcpServerSocket {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(2020);
        /*
            建立一个负责监听2020端口的服务器
         */
        Socket accept = serverSocket.accept();
        /*
            等待客户端接入服务器
         */
        OutputStream outputStream = accept.getOutputStream();
        InputStream inputStream = accept.getInputStream();

        outputStream.write("你好客户端，这里是服务器".getBytes());
        byte[] bytes = new byte[1024];
        int len = inputStream.read(bytes);
        String content = new String(bytes, 0, len, "UTF-8");
        System.out.println(content);

        accept.close();
        serverSocket.close();
    }
}

```

客户端
```java
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.net.Socket;

/**
 * @author BlackKnight
 */
public class TcpSocket {
    public static void main(String[] args) throws IOException {
        //Socket socket = new Socket("127.0.0.1",2020);
        Socket socket = new Socket();
        socket.connect(new InetSocketAddress("127.0.0.1",2020),10000);
        /*
            第一种创建方式会一直阻塞等待下去，直到和host:port建立了连接为止
            第二种创建方式可以解决这个问题，先创建一个Socket类，然后连接的同时设置一个响应超时时间
        */
        OutputStream outputStream = socket.getOutputStream();

        outputStream.write("你好啊服务器，我是客户端".getBytes());
        //创建输出流，向服务器发送数据

        InputStream inputStream = socket.getInputStream();
        byte[] bytes = new byte[1024];
        int len = inputStream.read(bytes);
        String content = new String(bytes,0,len,"UTF-8");
        /*
            通过指定的编码来将byte型数组解析为字符串
            bytes - 要解码为字符的字节
            offset - 要解码的第一个字节的索引
            length - 要解码的字节数
            charsetName - 解码方式
         */
        System.out.println(content);
        socket.close();
    }
}
```

# 实现多线程

服务器
```java
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @author BlackKnight
 */
public class TcpServerSocketMultithreading {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(2020);
        while (true){
            Socket socket = serverSocket.accept();
            Runnable r = new TcpServerSocketMultithreadingImpl(socket);
            Thread t = new Thread(r);
            t.start();
        }
    }
}
```
实现类
```java
import java.io.IOException;
import java.net.Socket;

/**
 * @author BlackKnight
 */
public class TcpServerSocketMultithreadingImpl implements Runnable {
    private Socket socket;

    TcpServerSocketMultithreadingImpl(Socket socket) throws IOException {
        this.socket = socket;
    }
    @Override
    public void run(){
        String s = "hello "+ socket.getInetAddress();
        try {
            socket.getOutputStream().write(s.getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```
