---
title: Java-API-4
date: 2019-08-13 11:46:16
tags: 
- Java
- JavaAPI
categories: 
- 后台
- Java
---

<center>
引言：

Date类

DateFormat类

Calendar类

</center>

<!--more-->

---

# Date

表示特定的瞬间，精确到毫秒，可以对日期和事件进行计算，计算完毕再讲毫秒转换为日期

## 包路径
```
java.util
```

## 构造方法
```java
public Date()
//分配 Date 对象并初始化此对象，以表示分配它的时间（精确到毫秒）。 

public Date(long date)
//分配 Date 对象并初始化此对象，以表示自从标准基准时间（称为“历元（epoch）”，即 1970 年 1 月 1 日 00:00:00 GMT）以来的指定毫秒数。
```

## 常用方法
```java
public long getTime()
//public long getTime()
```

## 示例代码
```java
System.out.println(System.currentTimeMillis());//系统的方法
Date d = new Date(); //构造一个日期类
System.out.println(d.getTime());//日期类的常用方法
```

# DateFormat(SimpleDateFormat)

对日期进行一个格式化，是一个抽象类

可以用来进行 **日期和文本** 之间的转化

DateFormat是一个抽象类，无法直接创建对象使用，可以使用DateFormat类的子类SimpleDateFormat来创建对象

## 包路径
```
java.text.DateFormat
java.text.SimpleDateFormat
```

## 构造方法

```java
public SimpleDateFormat(String pattern)
//参数：字符串类型的参数，要传递一个指定的模式
//年(y)月(M)日(d)时(H)分(m)秒(s)
//例如 "yyyy-MM-dd HH:mm:ss"
```
注意：模式中的字母不能改变，连接的符号可以改变

例如改为 "yyy年MM月dd日 HH:mm:ss"
## 常用方法

DateFormat的常用方法
```java

public String format(Date date)
//按照指定的模式，把日期转换为符合模式的字符串

public Date parse(String source)
//把符合模式的字符串转换为date日期

```

SimpleDateFormat的常用方法
```java
format(Date date, StringBuffer toAppendTo, FieldPosition pos) 
//将给定的 Date 格式化为日期/时间字符串，
//并将结果添加到给定的 StringBuffer。

parse(String text, ParsePosition pos)
//解析字符串的文本，生成 Date。
//声明了一个异常叫ParseException，
//如果字符串和构造方法的模式不同，则会抛出此异常
```
## 示例代码
```java
//这里要加一个throws抛出异常
SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH时mm分ss秒");
// 创建一个格式
Date date = new Date();
// 创建当前日期
System.out.println(sdf.format(date));// 按照格式解析日期对象
Date d = sdf.parse("2000年1月5日 00时00分00秒");
// 解析文本,文本格式不正确的话会报出异常
System.out.println(d);
```

# Calendar

日历类，也是一个抽象类，无法直接创建对象

它为特定瞬间与一组诸如 YEAR、MONTH、DAY_OF_MONTH、HOUR 等 日历字段之间的转换提供了一些方法，并为操作日历字段（例如获得下星期的日期）提供了一些方法。

## 包路径
```java
java.util
```

## 构造方法
```java
protected Calendar()
```
## 常用方法
```java
public static Calendar getInstance()
//使用默认时区和语言环境获得一个日历。返回的 Calendar 基于当前时间，使用了默认时区和默认语言环境。 

public int get(int field)
//返回给定日历字段的值

public void set(int field,int value)
//将给定的日历字段设置为给定值。

public abstract void add(int field,int amount)
//根据日历的规则，为给定的日历字段添加或减去指定的时间量

public final Date getTime()
//返回一个表示此 Calendar 时间值（从历元至现在的毫秒偏移量）的 Date 对象。 
```

## 示例代码

```java
Calendar c = Calendar.getInstance();// getInstance方法
System.out.println(c);//打印出一长串日历字段

//参数 int field :是一个日历类的字段，可以使用Calendar类的静态成员变量获取
int year = c.get(Calendar.YEAR);
int month = c.get(Calendar.MONTH);// 西方月份0-11
int date = c.get(Calendar.DATE);
//get方法
System.out.println(year +" "+ month +" "+ date);//2019 7 13

//set方法
c.set(Calendar.YEAR,9999);
c.set(Calendar.MONTH,9);
c.set(Calendar.DATE,9);
c.set(8888,8,8);//同时设置
System.out.println(c.get(Calendar.YEAR));// 8888

c.add(Calendar.YEAR,2);//add方法,增加两年
System.out.println(c.get(Calendar.YEAR));//8890

Date d = c.getTime();// getTime返回一个日期类
System.out.println(d);// Fri Sep 08 11:44:23 GMT+08:00 8890
```
