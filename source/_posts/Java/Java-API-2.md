---
title: Java-API-2
date: 2019-08-12 10:54:14
tags: 
- Java
- JavaAPI
categories: 
- 后台
- Java
---
<center>
引言：

String类

Arrays类

</center>

<!--more-->

-------

# String

String 类代表字符串。Java 程序中的**所有**字符串字面值（如 "abc" ）都作为此类的实例实现。 

字符串是常量；它们的值在创建之后不能更改。**字符串缓冲区**支持可变的字符串。因为 String 对象是不可变的，所以可以共享。 

## 包路径：
```
java.lang
```

## 构造方法：
```java
public String()
//初始化一个新创建的 String 对象，使其表示一个空字符序列。
//由于 String 是不可变的，所以无需使用此构造方法。 
public String(char[] value) //表示字符数组参数中当前包含的字符序列
public String(byte[] bytes)
//通过使用平台的默认字符集解码指定的 byte 数组，构造一个新的 String。
//还有一种直接创建的方式，使用类似于int关键字
```

## 常用方法：

比较方法
```java
public boolean equals(Object anObject)
//将此字符串与指定的对象比较。
//当且仅当该参数不为null，并且是与此对象表示相同字符序列的 String 对象时，结果才为 true。 

public boolean equalsIgnoreCase(String anotherString)
//不考虑大小写比较。
//如果两个字符串的长度相同，并且其中的相应字符都相等（忽略大小写），则认为这两个字符串是相等的。
```
属性获取方法
```java
public int length()//返回此字符串的长度

public boolean isEmpty()//长度为0时返回 true。 

public char charAt(int index)//返回指定索引处的 char 值。索引范围为从 0 到 length() - 1

public int indexOf(String str)//返回指定子字符串在此字符串中第一次出现处的索引，没有返回-1
```
裁剪方法
```java
public String concat(String str)//将指定字符串连接到此字符串的结尾。 
//如果参数字符串的长度为 0，则返回此 String 对象。否则，创建一个新的 String 对象

public String substring(int index);//从参数位置一直到字符串末尾，返回新字符串

public String(intbegin,intend);//从begin开始到end,左闭右开区间。
```
转换方法
```java
public char[] toCharArray();//将当前字符串拆分为字符数组作为返回值

public byte[] getBytes();//获得当前字符串的底层的字节数组

public String replace(CharSequence oldString,CharSequence newString);
//将所有的老字符串换成新的字符串
```
分割方法
```java
public String[] split(String regex);
//按参数的规则，将字符串且分为若干部分（参数是一个正则表达式,不能切割英文句点"."，要写"."要加转义符反斜杠"\\."）
```

## 示例代码：

构造方法
```java
String str1 = "hello";//直接构造
System.out.println(str1);// hello

String str2 = new String();//空参构造一个空字符串
System.out.println(str2);// ''空

char[] charArray = {'a','w','c'};
String str3 = new String(charArray);//用字符数组构造
System.out.println(str3);//awc

byte[] byteArray = {85,49,35};
String str4 = new String(byteArray);//用ASCII码构造
System.out.println(str4);//U1#
```
常用方法:

比较方法
```java
String str1 = "hello";
String str2 = "hello";
char[] arr = {'h','e','l','l','o'};
String str3 = new String(arr);
String str4 = "HellO";
System.out.println(str1==str2);//true
//直接使用等号比较，比较的是地址，又因为相同字符串的地址会归在一起，所以为true
System.out.println(str1==str3);//false 与str3此时地址就不一样了
System.out.println(str1.equals(str2));//true 直接比较内容
System.out.println(str1.equals(str3));//true
System.out.println(str1.equalsIgnoreCase(str4));//true
```
属性获取方法
```java
System.out.println(str1.length());//5   长度获取方法
System.out.println(str1.charAt(3));//l  获取元素方法
System.out.println(str1.indexOf("h"));//0  查找索引方法
```
裁剪方法
```java
String str1 = "hello";
String str2 = "hello";

str1.concat(str2);//hellohello

"你好".concat(str1);//你好hello

str1.substring(2);//llo 

str1.substring(2,4); //ll
```

转换方法
```java
String str1 = "hello";

char[] arr1 = str1.toCharArray(); //拆分当前字符串为数组
byte[] arr2 = str1.getBytes();  // 获得当前字符串的底层数组
String str2 = str1.replace('l','w');
// hewwo 新字符代替旧字符
```

分割方法
```java
String str1 = "he,l,lo";
String[] str2 = str1.split(",");// 用,对其进行分割
for (int i = 0; i < str2.length; i++) {
    System.out.println(str2[i]);// 分成he  l  lo
}
```

## 字符串常量池

在java虚拟机堆中，字符串有一个专门的常量池

**直接写上双引号的字符串，就在字符串常量池中**

**new创建的字符串不在常量池内**

对于引用类型来说，等于号是对地址进行的比较

特点：
1. 字符串的内容不可以改变
2. 字符串内部是可以共享使用的，相同加了双引号的字符串，如果内容都相同，他的地址也是来自一块地址的
3. 字符串的底层原理是byte字节数组


# Arrays

用来操作数组（比如排序和搜索）的各种方法

## 包路径
```
java.util
```

## 构造方法

所有的数组都可以使用这个类的方法

## 常用方法
```java
public static String toString(int[] a)
//把数组转换为字符串，数组不限定类型，有多种重载函数

public static void sort(int[] a)
//排序：数字按从小到大排序，字母按字母升序
```

## 示例代码

```java
int[] arr = {3,8,2,1};
String str = Arrays.toString(arr);// 转换为字符串
System.out.println(str);// [3,8,2,1]

Arrays.sort(arr);//从小到大排序，改变了原数组的顺序
System.out.println(Arrays.toString(arr));//[1,2,3,8]
```