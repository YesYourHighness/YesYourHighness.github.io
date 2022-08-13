---
title: Java-IO-缓冲流
date: 2019-08-23 21:50:05
tags: 
- Java
- JavaIO
categories: 
- 后台
- Java
---

<center>
引言：

高效读写的缓冲流

</center>

<!--more-->

---


# 缓冲流

高效读写，对四种基本流的增强

流程：

创建后 -> JVM -> OS -> 数据的字节

但是区别在于，每次将不再是一个一个返回，放在一个数组内部，一次性返回，增加了效率

分类：
- 字节缓冲流:`BufferedInputStream`,`BufferedOutputStream`
- 字符缓冲流:`BufferedReader`,`BufferedWriter`


# 字节缓冲流

## BufferedOutputStream
继承了`OutputStream`,

子类继承父类，可以使用父类的方法

### 包路径

```java
java.io
```

### 构造函数

```java
BufferedOutputStream(OutputStream out) 
//创建一个新的缓冲输出流，以将数据写入指定的底层输出流。 

BufferedOutputStream(OutputStream out, int size) 
//创建一个新的缓冲输出流，以将具有指定缓冲区大小的数据写入指定的底层输出流。 

//第一个参数：字节输出流
//第二个参数：指定缓冲流内部缓冲区的大小，不能默认
```
步骤：
1. 创建`FileOutputStream`对象，构造方法绑定目的地
2. 创建`BufferedOutputStream`对象，构造方法中传递`FileOutputStream`对象，提高`FileOutputStream`对象效率
3. 使用`BufferedOutputStream`对象中的方法`write`，把数据写入到内部缓冲区中
4. 使用`BufferedOutputStream`对象中的flush方法，把内部缓冲区的数据刷新到文件
5. 释放资源(会先调用flush方法刷新数据)

```java
FileOutputStream fos = new FileOutputStream("D:\\a.txt");
BufferedOutputStream bos=  new BufferedOutputStream(fos);
bos.write("写入数据到内部缓冲区".getBytes());
bos.flush();
bos.close();
```

## BufferedInputStream

继承了`InputStream`,

子类继承父类，可以使用父类的方法

### 包路径
```
java.io
```
### 构造函数
```java
BufferedInputStream(InputStream in) 
//创建一个 BufferedInputStream 并保存其参数，即输入流 in，以便将来使用。 

BufferedInputStream(InputStream in, int size) 
//创建具有指定缓冲区大小的 BufferedInputStream 并保存其参数，即输入流 in，以便将来使用。 

//第一个参数：字节输入流
//第二个参数：执行缓冲流内部缓冲区大小，不能默认
```
步骤：
1. 创建`FileInputStream`对象，构造方法中绑定要读取的数据源
2. 创建`BufferedInputStream`对象，构造函数绑定`FileInputStream`对象
3. 使用`BufferedInputStream`对象中的方法`read`，读取文件
4. 释放资源
```java
FileInputStream fis = new FileInputStream("D:\\a.txt");
BufferedInputStream bis = new BufferedInputStream(fis);
//一个一个字节读取
int len = 0;
while ((len = bis.read()) != -1) {
    System.out.println(len);
}
bis.close();

//每次读取1kb
byte[] bytes = new byte[1024];
int len1 = 0;
while ((len1 = bis.read(bytes)) != -1) {
    System.out.println(new String(bytes, 0, len1));
}
```

## 字节缓冲流复制文件
复制文件速度十分快，远远快于普通的流的速度
```java
    //字节缓冲流复制文件
    FileInputStream fis = new FileInputStream("D:\\a.txt");
    FileOutputStream fos = new FileOutputStream("D:\\b.txt");
    BufferedInputStream bis = new BufferedInputStream(fis);
    BufferedOutputStream bos = new BufferedOutputStream(fos);
    int len = 0;
    while ((len = bis.read())!=-1){
        bos.write(len);
    }
    bos.close();
    bis.close();
```

# 字符缓冲流

将文本写入字符输出流，缓冲各个字符，从而提供单个字符、数组和字符串的高效写入。 


## BufferedWriter
字符缓冲输出流，继承父类的所有方法

### 包路径

```
java.io
```

### 构造方法


```java
BufferedWriter(Writer out) 
//创建一个使用默认大小输出缓冲区的缓冲字符输出流

BufferedWriter(Writer out, int sz) 
//创建一个使用给定大小输出缓冲区的新缓冲字符输出流 

//参数：
//第一个传递FileWriter，缓冲流给FileWriter增加一个缓冲区，提高FileWriter的写入效率
//第二个设定缓冲区的大小，不写会有默认的大小
```

### 常用方法
```java
void newLine() //throws IOException写入一个行分隔符

void write(char[] cbuf,int off,int len)throws IOException
//写入字符数组的某一部分

void write(String s,int off,int len)throws IOException
//写入字符串的某一部分。 

public void flush()throws IOException
//刷新该流的缓冲。 

public void close()throws IOException
//关闭此流，但要先刷新它。
```
步骤：
1. 创建字符缓冲输出流对象，构造方法中传递字符输出流
2. 调用字符缓冲输出流中的方法write，把数据写入到内存缓冲区中
3. 调用字符缓冲输出流中的方法flush，把缓冲区数据刷新到文件
4. 释放资源

```java
BufferedWriter bw = new BufferedWriter(new FileWriter("D:\\a.txt"));
for(int i = 0;i<10;i++){
    bw.write("你好");
    bw.newLine();
}
bw.flush();
bw.close();
```

## BufferedReader

从字符输入流中读取文本，缓冲各个字符，从而实现字符、数组和行的高效读取。 

### 包路径
```
java.io
```

### 构造方法
```java
BufferedReader(Reader in) 
//创建一个使用默认大小输入缓冲区的缓冲字符输入流。

BufferedReader(Reader in, int sz) 
//创建一个使用指定大小输入缓冲区的缓冲字符输入流。 

//参数：
//第一个参数绑定一个FileReader类
//第二个参数设定缓冲区大小
```

### 常用方法
```java
public String readLine()throws IOException
//读取一个文本行。
//通过下列字符之一即可认为某行已终止：
//换行 ('\n')、回车('\r')或回车后直接跟着换行

public int read()throws IOException
//读取单个字符。 

public int read(char[] cbuf,int off,int len)throws IOException
//将字符读入数组的某一部分。

public void close()throws IOException
//关闭流并在关闭前刷新数据
```
步骤：
1. 创建字符缓冲输入流对象，构造方法中传递字符输入流
2. 使用字符缓冲输入流对象的方法`read/read line`读取文本
3. 释放资源

