---
title: Java-IO-字节流
date: 2019-08-21 22:03:51
tags: 
- Java
- JavaIO
categories: 
- 后台
- Java
---

<center>
引言: 字节流
</center>

<!--more-->

---


# IO
## 什么是IO流

- `input`:输入——数据从硬盘到内存
- `output`:输出——数据从内存到硬盘
- 流：数据

## IO的分类

根据数据流向可以分为**输入流**和**输出流**

输入流：把数据从`其他设备`读取到`内存`中的流

输出流：把数据从`内存`写出到`输出设备`上的流

格局数据的类型分为:**字节流**和**字符流**


## 一切皆为字节

明确一个概念：

一切文本、视频、图片、视频在存储时，都是以二进制数字的形式保存的

## 字节输出流`OutputStream`

此抽象类是表示输出字节流的所有类的超类。

输出流接受输出字节并将这些字节发送到某个接收器


### 常用方法
```java
public void close()throws IOException
//关闭此输出流并释放与此流有关的所有系统资源

public void flush() throws IOException
//刷新此输出流并强制写出所有缓冲的输出字节

public abstract void write(int b)throws IOException
//一共有三个write，作用相同
//将指定的字节写入此输出流
```

### 子类
```java
ByteArrayOutputStream
//向数组写数据

FileOutputStream,
//向文件写数据
//重点介绍

FilterOutputStream,
//带过滤器的文件写入

ObjectOutputStream, 
//向对象写数据

OutputStream, 
//其他包内的流

PipedOutputStream 
//管道流
```

## FileOutputStream

### 包路径
```
java.io
```

### 构造方法
```java
FileOutputStream(File file) 
// 创建一个向指定 File 对象表示的文件中写入数据的文件输出流
//参数是一个文件 
 
FileOutputStream(String name) 
//创建一个向具有指定名称的文件中写入数据的输出文件流
//参数是一个路径

```
作用：
1. 创建一个`FileOutputStream`对象
2. 会根据构造方法中传递的文件/文件路径，创建一个空的文件
3. 会把`FileOutputStream`对象指向创建好的文件


### 写数据到文件
原理：内存->文件

Java程序 --> JVM --> OS(操作系统) --> OS调用写数据的方法 --> 把数据写入到文件中


步骤：
1. 创建一个FileOutputStream对象，构造方法中传递写入数据的目的地
2. 调用FileOutputStream对象中的方法write，把数据写入到文件中
3. 释放资源（流会占用一定的内存，使用完毕要把内存清空，提高程序的效率）

```java
public static void main(String[] args) throws IOException {
    FileOutputStream fos = new FileOutputStream("D:\\a.txt");
    fos.write(97);
    fos.close();
    //文件就自动创建并且写入了一个 a
}
```

### 文件存储和记事本打开原理

- 存储

写入数据会把十进制的整数转化为二进制整数，

`fos.write(97);`会把`97`变为`1100001`

- 记事本

任意的文本编辑器（记事本），

在打开文件的时候都会查询编码表，

把字节转换为字符表示，会把`97`变为`a`


### 字节输出流写多个字节的方法

```java
FileOutputStream fos = new FileOutputStream("D:\\a.txt");
byte[] bytes = {-49,-99,-28,120};
//如果第一个字节是正数(0-127),那么会查询ASCII码表显示
//如果第一个字节是负数，会与第二个字节组成一个中文显示
fos.write(bytes);
fos.close();
```
选定输入几个字节
```java
FileOutputStream fos = new FileOutputStream("D:\\a.txt");
byte[] bytes = {49,99,28,120};
fos.write(bytes,1,2);
//int off 开始的索引
//int len 写几个字节
fos.close();
```
输入字符串
```java
FileOutputStream fos = new FileOutputStream("D:\\a.txt");
//String有getBytes()的方法，可以把字符串转换为字节数组
fos.write("你好啊".getBytes());
fos.close();
```

### 数据的追加续写

这就用到了另外两个FileOutputStream的构造方法
```java
FileOutputStream(File file, boolean append) 
//创建一个向指定File对象表示的文件中写入数据的文件输出流
          
FileOutputStream(String name, boolean append) 
//创建一个向具有指定name的文件中写入数据的输出文件流。

//第一个参数都是文件的位置，第二个参数是是否追加写的开关，true的话不会覆盖原文件，false则会覆盖原文件
```

```java
FileOutputStream fos = new FileOutputStream("D:\\a.txt",true);
//String有getBytes()的方法，可以把字符串转换为字节数组
fos.write("\r\n".getBytes());
fos.write("你好啊".getBytes());
//换行:
// windows:\r\n
// Linux:/n
// mac: /r
fos.close();
```

## 字节输入流`InputStream`
此抽象类是表示字节输入流的所有类的超类。 



### 包路径
```
java.io
```

### 子类
```java
AudioInputStream
//读取音频

ByteArrayInputStream, 
//读取字节数

FileInputStream, 
//读取文件
//重点研究

FilterInputStream, 
//带过滤器

ObjectInputStream, 
//读取对象

PipedInputStream, 
//读取管道

SequenceInputStream, 
//读取队列

StringBufferInputStream 
//读取字符串缓冲区

```

### 常用方法
```java
public abstract int read()throws IOException
//从输入流中读取数据的下一个字节,返回 0 到 255 范围内的 int 字节值

public abstract int read() throws IOException
//从输入流中读取一定数量的字节，并将其存储在缓冲区数组 b 中

public void close() throws IOException
//关闭此输入流并释放与该流关联的所有系统资源
```

## FileInputStream

### 构造方法
```java
FileInputStream(File file) 
//通过打开一个到实际文件的连接来创建一个FileInputStream
//该文件通过文件系统中的 File 对象 file 指定。

FileInputStream(String name) 
//通过打开一个到实际文件的连接来创建一个 FileInputStream，
//该文件通过文件系统中的路径名 name 指定
```

作用：
1. 创建一个`FileInputStream`对象
2. 会把`FileInputStream`对象指定到构造方法的位置中去读取文件

原理：

Java程序 -> JVM -> OS -> OS读取数据的方法 -> 读取文件

### 常用方法

```java
public int read()throws IOException
//从此输入流中读取一个数据字节,读取完成后再次调用此方法，会读取下一个数据
//返回-1表示读取完毕

public int read(byte[] b)throws IOException
//从此输入流中将最多 b.length 个字节的数据读入一个 byte 数组中

public int read(byte[] b,int off,int len)throws IOException
//从此输入流读取任意字节的数据
```

步骤：
1. 创建FileInputStream对象，构造方法中绑定要读取的数据源
2. 使用`FileInputStream`中的`read`方法读取文件
3. 释放资源



```java
FileInputStream fis = new FileInputStream("D:\\a.txt");
int len = fis.read();
//读取第一个字符
System.out.println(len);
int len2 = fis.read();
System.out.println(len2);
//读取第二个字符
fis.close();
```
读取未知文件，使用while循环，固定写法
```java
FileInputStream fis = new FileInputStream("D:\\a.txt");
int len = 0;
//这个变量必须定义，否则读取的指针在判断时会改变
//记录读取的字符
while ((len = fis.read())!=-1){
    System.out.print((char)len);
}
fis.close();
```
---
原理:

创建构造函数之后，会创建一个指针指向文件的第一个字节，每当运行`read()`方法，指针就指向下一个

当`read()`方法返回-1时，读取将会结束

---

一次读取多个字节的方法

使用第二个构造函数

注意：
1. byte[]设定了一次读取的个数，起到缓冲作用，存储每次读取到的多个字节；数组的长度一般设定为1024或者1024的整数倍
2. `read`方法的返回值int是每次读取的有效元素的长度
```java
FileInputStream fis = new FileInputStream("D:\\a.txt");
byte[] bytes = new byte[2];
int len = fis.read(bytes);
//返回数组的长度
System.out.println(len);
System.out.println(Arrays.toString(bytes));
fis.close();
```
---
原理：

一次读取多个字节，

构造函数创建后，指针指向第一个字节

在此后的read方法中，会一次读取数组长度的字节

之后指向数组长度之后的那个字节

---

读取固定写法
```java
FileInputStream fis = new FileInputStream("D:\\a.txt");
byte[] bytes = new byte[1024];
int len = 0;
while ((len = fis.read(bytes))!=-1){
System.out.println(new String(bytes,0,len));
}
fis.close;
```


## 文件复制

原理：

读入再输出即可

```
//复制一个文件
FileInputStream fis = new FileInputStream("D:\\a.txt");
//读入
FileOutputStream fos = new FileOutputStream("D:\\a\\a.txt");
//输出
int len = 0;
while ((len=fis.read())!=-1) {
    fos.write(len);
}
fos.close();//先关闭输出的
fis.close();
```