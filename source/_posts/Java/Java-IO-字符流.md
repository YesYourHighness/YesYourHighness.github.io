---
title: Java-IO-字符流
date: 2019-08-21 22:03:38
tags: 
- Java
- JavaIO
categories: 
- 后台
- Java
---

<center>
引言：

字符流
</center>

<!--more-->

---

# 字符流
为了解决读取中文可能会出现的显示不了完整的字符的问题，java提供了字符流类

## Reader

用于读取字符流的抽象类,是一个抽象类所以我们得使用子类

子类很多，我们重点学习`InputStreamReader`类下的`FileReader`这个类
### 包路径
```
java.io
```

### 常用方法
```java
public int read()throws IOException
//读取单个字符

public int read(char[] cbuf)throws IOException
//将字符读入数组

public abstract void close()throws IOException
//关闭该流并释放与之关联的所有资源
```
## Writer

同样有很多子类，我们主要关注`FileWrite`这个子类
### 包路径
```
java.io
```

### 常用方法
```java
public void write(int c)throws IOException
//写入单个字符

public void write(char[] cbuf)throws IOException
//写入字符数组。 

public abstract void write(char[] cbuf,int off,int len)throws IOException
//写入字符数组的某一部分。 
```



## FileReader

用来读取字符文件的便捷类，
### 包路径
```
java.io
```

### 构造方法
```java
public FileReader(File file)throws FileNotFoundException
//在给定从中读取数据的 File 的情况下创建一个新 FileReader。 
           
public FileReader(String fileName)throws FileNotFoundException
//在给定从中读取数据的文件名的情况下创建一个新 FileReader。 
```
构造方法的作用：
1. 创建一个FileReader对象
2. 会把FileReader对象指向要读的文件

### 读取数据
步骤：
1. 创建`FileReader`对象，构造方法中绑定要读取的数据源
2. 使用`FileReader`对象中的方法`read`读取文件
3. 释放资源

```java
FileReader fr = new FileReader("D:\\a.txt");
int len = 0;
while ((len=fr.read()) != -1){
System.out.print((char)len);
}
fr.close();
```
数组也是可以的
```java
FileReader fr = new FileReader("D:\\a.txt");
int len = 0;
char[] cs = new char[1024];
while ((len=fr.read(cs)) != -1){
System.out.print(new String(cs,0,len));
//选择转化多少为字符串
}
fr.close();
```

## FileWrite
把内存中的字符数据写入到文件中

### 包路径
```java
java.io
```

### 构造方法
```java
public FileWriter(File file)throws IOException
//参数：写入数据的目的地

public FileWriter(String fileName)throws IOException
//根据给定的文件名构造一个 FileWriter 对象。

```
步骤:
1. 创建`FileWriter`对象，构造方法中绑定要写入数据的目的地
2. 使用`FileWriter`中的方法`write`，**把数据写入到内存缓冲区中**（字符转换为字节的过程）
3. 使用`FileWriter`中的方法`flush`，把内存缓冲区中的数据刷新到文件中
4. 释放资源(会先把内存缓冲区中的数据刷新到文件中)

注意：
与其他输出方法不同，writer要走内存缓冲区
```java
FileWriter fw = new FileWriter("D:\\a.txt");
fw.write(88);
fw.flush();
fw.close();
```

### `close`与`flush`区别
两个方法都可以把数据刷新到文件中

- flush : 刷新缓冲区，流对象可以继续使用
- close : 先刷新缓冲区，然后通知系统释放资源，流对象不可以再被使用了

### 写出其他数据
可以写字符数组，写字符数组的一部分，写字符串，还有字符串的某一部分

```java
FileWriter fw = new FileWriter("D:\\a.txt");
char[] cs = {'a','w','w'};
String s = "你好啊";
fw.write(cs);
//写字符数组
fw.write(cs,0,1);
//从第一个开始写，写一个
fw.write(s);
//写一个字符串
fw.write(s,0,1);
//从第一个字符串开始写，写一个
fw.close();
```
### 续写与换行
同字节流
```java
FileWriter fw = new FileWriter("D:\\a.txt",true);
//续写
fw.write("\n\r");
//windows换行\r\n
//Linux换行\n
//mac换行\r
fw.write("later equals never");
fw.close();
```


# IO流异常抛出的处理
一直我们都是throws丢给JVM来处理，但是我们可以使用try\catch\finally来处理异常

```java
public static void main(String[] args) {
    FileWriter fw = null;
    try {
        fw = new FileWriter("w:\\a.txt", true);
        fw.write("\n\r");
        fw.write("later equals never");
    } catch (IOException e) {
        System.out.println("出错啦");
    } finally {
        if (fw != null) {
            //判断fw是否为空
            try {
                fw.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

- JDK7新特性
        
    在try的后边可以增加一个()，在括号中可以定义**流对象**，那么这个流对象的作用域就在try有效，try中的代码执行完毕，会把流对象自动释放，不用写finally

    格式：
    ```java
        try(定义流对象;定义流对象...){
            //可能会产生异常代码
        }catch(异常类变量 变量名){
            //异常的处理逻辑
        }
    ```
```java
try (
        FileInputStream fis = new FileInputStream("D:\\a.txt");
        FileOutputStream fos = new FileOutputStream("D:\\a.txt");
) {
    int len = 0;
    while ((len = fis.read()) != -1) {
        fos.write(len);
    }
    } catch (IOException e) {
        e.printStackTrace();
}
```
- JDK9新特性
    
    try的前面我们可以定义流对象，try后面的()中，可以直接引入流对象的名称(变量名)

    在try代码执行完毕之后，流对象也可以释放掉，不用写finally
    
    格式:
    ```java
    A a = new A();
    B b = new B();
    try(a,b){
        //可能会产生异常代码
    }catch(异常类变量 变量名){
        //异常的处理逻辑
    }
    ```
