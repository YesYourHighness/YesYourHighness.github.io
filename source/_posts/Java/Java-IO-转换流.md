---
title: Java-IO-转换流
date: 2019-08-23 21:50:47
tags: 
- Java
- JavaIO
categories: 
- 后台
- Java
---

<center>
引言：

转换流
</center>

<!--more-->

---


# 字符编码与字符集

- 字符编码：

计算机只能识别二进制，

电脑上的任何软件都是以二进制的1和0存储的，将字符存储到计算机中，称为**编码**，

将二进制解析显示出来称为**解码**。

**字符编码就是两者之间对应的转换规则**

- 字符集
是一个系统所有字符的集合，包含各国文字等等

常见的字符集有：

```
ASCII编码-->ASCII字符集
GBK编码-->GBK字符集
UTF-8编码-->Unicode字符集
UTF-16编码-->Unicode字符集
UTF-32编码-->Unicode字符集
```

## 编码引出的问题

在IDEA中，使用UTF-8,日常使用没有问题，FileReader默认也是读取UTF-8

但是windows中有GBK编码，这样使用FileReader读取GBK的文件，就会产生乱码


# 转换流

## InputStreamReader
**字节流通向字符流的桥梁：**

它使用指定的charset读取字节并将其解码为字符。

它使用的**字符集可以由名称指定或显式给定，或者可以接受平台默认的字符集**

### 构造方法
```java
InputStreamReader(InputStream in) 
//创建一个使用默认字符集的 InputStreamReader。 

InputStreamReader(InputStream in, String charsetName) 
//创建使用指定字符集的 InputStreamReader。
//第二个参数写入指定的编码表名称，默认使用UTF-8，不区分大小写
```
### 常用方法
```java
public String getEncoding()
//返回此流使用的字符编码的名称。 

//其他方法同FileReader
```
步骤：

1. 创建一个`InputStreamReader`对象，构造方法中传递字节输入流和指定的编码表名称
2. 使用`InputStreamReader`对象中的方法read读取文件
3. 释放资源

示例代码
```java
InputStreamReader osr = new InputStreamReader(new FileInputStream("D:\\a.txt"),"UTF-8");
int len = 0;
System.out.println(osr.getEncoding());
//打印出UTF8
while ((len = osr.read())!=-1){
    System.out.println((char)len);
}
osr.close();
```

## OutputStreamReader
**字符流通向字节流的桥梁**：可使用指定的 charset 将要写入流中的字符编码成字节。

它使用的字符集可以由名称指定或显式给定，否则将接受平台默认的字符集。 

### 构造方法
```java
OutputStreamWriter(OutputStream out) 
//创建使用默认字符编码的 OutputStreamWriter。 


OutputStreamWriter(OutputStream out, String charsetName) 
//创建使用指定字符集的 OutputStreamWriter。
//第二个参数写入指定的编码表名称，默认使用UTF-8，不区分大小写
```

步骤：
1. 创建`OutputStreamWriter`对象，构造方法中传递字节输出流和指定的编码表名称
2. 使用`OutputStreamWrirer`对象中的`write`方法，把字符转换为字节存储缓冲区中
3. 使用`OutputStreamWriter`的方法`flush`，把内存缓冲区中的字节刷新到文件中
4. 释放资源
示例代码

```java
OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("D:\\a.txt"),"UTF-8");
//构造函数中指定所要的编码表
osw.write("你好");
osw.flush();
osw.close();
```
