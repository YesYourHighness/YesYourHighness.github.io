---
title: Java-API-5
date: 2019-08-13 15:52:19
tags: 
- Java
- JavaAPI
categories: 
- 后台
- Java
---

<center>
引言：

System类

StringBuilder类
</center>

<!--more-->

----

# System

System 类包含一些有用的类字段和方法。

它不能被实例化。 

在 System 类提供的设施中，有标准输入、标准输出和错误输出流；

对外部定义的属性和环境变量的访问；加载文件和库的方法；还有快速复制数组的一部分的实用方法。 

## 包路径
```
java.lang
```

## 构造方法

不能实例化


## 常用方法
```java
public static long currentTimeMillis()
//返回以毫秒为单位的当前时间

public static void arraycopy(Object src,int srcPos,Object dest,int destPos,int length)
//从指定源数组中复制一个数组，复制从指定的位置开始，到目标数组的指定位置结束
//五个参数:src(源数组),srcPos(源数组的起始位置),dest(目标数组),destPos(目标数组中的起始位置),length(要复制的数组元素的数量)
```

# StringBuilder
String类是一个常量，他们的值在创建以后不能更改

而字符串缓冲区支持可变的字符串——StringBuider类

可以提高字符串的操作效率

## String类和StringBuilder类的区别

- String类

    字符串是一个常量，他们的值在创建之后不能再更改，字符串的底层是一个被final修饰的数组，不能改变，是一个常量`private final byte[] value`
    
    比如说进行一个字符串的加减`String s = "a" + "b" + "c" = "abc"`
    
    在这个过程中，会出现五个字符串
    "a" "b" "c" "ab" "abc"
    
    这样大大的占用了空间
    
- StringBuilder类
  
    是一个字符串的缓冲区，可以提高字符串的操作效率，可以看成一个长度可以变化的字符串，底层也是一个数组，但是没有被final修饰
    `byte[] value = new byte[16]`

    StringBuilder类在内存中始终是一个数组，占用空间少，效率高
    
    如果超出了StringBuilder的容量，就会自动的扩容

## 包路径

```
java.lang
```

## 构造函数

```java
public StringBuilder()
//构造一个不带任何字符的字符串生成器，其初始容量为 16 个字符。 

public StringBuilder(String str)
//构造一个字符串生成器，并初始化为指定的字符串内容。该字符串生成器的初始容量为 16 加上字符串参数的长度。 
```

## 常用方法

```java
public StringBuilder append(boolean b)
//参数可以是任意类型，有多个重载函数
//将 boolean 参数的字符串表示形式追加到序列,
//参数将被转换成字符串,将所得字符串中的字符追加到此序列。 

public String toString()
//返回此序列中数据的字符串表示形式
```

## 示例代码

构造方法
```java
StringBuilder builder1 = new StringBuilder();//空参构造
System.out.println(builder1);// 空
StringBuilder builder2 = new StringBuilder("abc");//有参构造
System.out.println(builder2);//abc
```

常用方法
```java
//使用append方法象棋中添加数据,
// append返回值是一个this,即返回调用方法的对象，
// 所以我们使用append方法不需要接收返回值
builder1.append(15);//int类型
builder1.append("你好");//字符串
System.out.println(builder1);//15你好
//可以链式编程
builder1.append(2019).append(true).append("链式编程");
System.out.println(builder1);//15你好2019true链式编程
```
```java
//StringBuilder类可以和String相互转换
//String->StringBuilder可以使用StringBuilder方法
String str1 = builder1.toString();
System.out.println(str1);//15你好2019true连式编程
//StringBuilder->String使用StringBuilder方法
String str2 = "字符串";
StringBuilder builder3 = new StringBuilder(str2).append("world");
System.out.println(builder3);//字符串world
```
