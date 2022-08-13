---
title: Java反射_Class
date: 2020-02-29 23:00:31
tags:
- 反射
categories:
- Java
- 反射
---

<center>
引言：反射机制
</center>
<!-- more -->

# 反射

反射是Java中比较高阶的技巧，理解了反射对理解JVM、使用Spring框架都有好处

> 反射就是Reflection，Java的反射是指程序在运行期可以拿到一个对象的所有信息。

# Class类
先来看一段官方API
> Instances of the class Class represent classes and interfaces in a running Java application.
（Class实例代表正在运行的类和接口）
An enum is a kind of class and an annotation is a kind of interface.
（枚举是类、注解是接口）
Every array also belongs to a class that is reflected as a Class object that is shared by all arrays with the same element type and number of dimensions. 
（每个数组也属于一个反映为Class对象的类，该类对象由具有相同元素类型和维数的所有数组共享）
The primitive Java types (boolean, byte, char, short, int, long, float, and double), and the keyword void are also represented as Class objects.
（Java的原始数据类型以及void关键字也表现为Class对象）

巴比巴卜、屋里哇啦说了些什么呢？

意思是全部的Java数据类型，只要一运行，都**可以**是Class的对象。

请注意，**一个`Class`对象实际上表示的是一个类型**，**而这个类型未必一定是一种类**。

例如， `int` 不是类，但 `int.class` 是一个 Class 类型的对象。 

在JVM第一次读取到一个`class`时，会把它加载到内存，而且会为他创建一个`Class`类型的实例（注意是Class还是class，不要搞错了！！）

## 创建
> 官方API：Class has no public constructor. Instead Class objects are constructed automatically by the Java Virtual Machine as classes are loaded and by calls to the defineClass method in the class loader.

**Class没有公有的构造方法，因为它的创建是当class加载时，通过类加载器的`defineClass`方法，由JVM自动创建的。**

> 上文中提到：请注意，**一个`Class`对象实际上表示的是一个类型**，**而这个类型未必一定是一种类**。

JVM持有的每个`Class`实例都指向一个`（class或interface）`：
```
┌───────────────────────────┐
│      Class Instance       │──────> String
├───────────────────────────┤
│name = "java.lang.String"  │
└───────────────────────────┘
┌───────────────────────────┐
│      Class Instance       │──────> Random
├───────────────────────────┤
│name = "java.util.Random"  │
└───────────────────────────┘
┌───────────────────────────┐
│      Class Instance       │──────> Runnable
├───────────────────────────┤
│name = "java.lang.Runnable"│
└───────────────────────────┘
```
每个`Class`都详细的描述了这个`class`的信息
```
┌───────────────────────────┐
│      Class Instance       │──────> String
├───────────────────────────┤
│name = "java.lang.String"  │
├───────────────────────────┤
│package = "java.lang"      │
├───────────────────────────┤
│super = "java.lang.Object" │
├───────────────────────────┤
│interface = CharSequence...│
├───────────────────────────┤
│field = value[],hash,...   │
├───────────────────────────┤
│method = indexOf()...      │
└───────────────────────────┘
```
**`JVM`为每个加载的class创建了对应的Class实例，并在实例中保存了该class的所有信息，包括类名、包名、父类、实现的接口、所有方法、字段等，因此，如果获取了某个Class实例，我们就可以通过这个Class实例获取到该实例对应的class的所有信息。**

反射是什么呢？
反射就是通过`Class`实例来获取`class`信息

## 使用Class
虽然我们没有创建`Class`的权利，但是我们可以调用它

[法一]：通过类名
```java
Class cls = String.class;
```
[法二]：通过实例
```java
String str = "你好";
Class cls = str.getClass();
```
[法二]：通过调用`forName()`方法，这个方法需要知道包路径（但是这个方法的参数只能填类或者接口，否则会抛出异常）
```java
Class.forName("java.lang.String");
```
JVM对于相同类型的类只会为它创建一个`Class`，所以上述三个方法会调取到同一个`Class`

### `Class`实例比较和`instanceof`区别
```java
Integer i = new Integer(1);
System.out.println(i instanceof Integer);//true
System.out.println(i instanceof Number);//true
```
`instanceof`判定子类是父类的也是祖父类的实例
```java
System.out.println(i.getClass() == Integer.class);//true
System.out.println(i.getClass() == Number.class);//不可比较，编译报错
```
`Class`实例只能使用`==`比较，并且比较的两个必须相同类型（在参考资料中，最后这一个比较会返回false，而不是报错，而我实测编译报错，版本为JDK1.8）

## 获取信息

反射的目的就是调用`Class`来获取`class`的信息

Class中有大量的方法，我们写一个如下的函数
```java
static void printClassInfo(Class cls) {
    System.out.println("Class name: " + cls.getName());
    System.out.println("Simple name: " + cls.getSimpleName());
    if (cls.getPackage() != null) {
        System.out.println("Package name: " + cls.getPackage().getName());
    }
    System.out.println("is interface: " + cls.isInterface());
    System.out.println("is enum: " + cls.isEnum());
    System.out.println("is array: " + cls.isArray());
    System.out.println("is primitive: " + cls.isPrimitive());
}
```
当我们输入参数为`String.class`（一个类）
```
Class name: java.lang.String
Simple name: String
Package name: java.lang
is interface: false
is enum: false
is array: false
is primitive: false
```
当我们输入参数为`Runnable.class`（一个接口）
```
Class name: java.lang.Runnable
Simple name: Runnable
Package name: java.lang
is interface: true
is enum: false
is array: false
is primitive: false
```
当我们输入参数为`java.time.Month.class`（一个枚举类）
```
Class name: java.time.Month
Simple name: Month
Package name: java.time
is interface: false
is enum: true
is array: false
is primitive: false
```
当我们输入参数为`String[].class`（一个数组）
```
Class name: [Ljava.lang.String;
Simple name: String[]
is interface: false
is enum: false
is array: true
is primitive: false
```
当我们输入参数为`int[].class`（一个数组）
```
Class name: [I
Simple name: int[]
is interface: false
is enum: false
is array: true
is primitive: false
```
当我们输入参数为`int.class`（一个基本类型）
```
Class name: int
Simple name: int
is interface: false
is enum: false
is array: false
is primitive: true
```
当我们输入参数为`void.class`（一个基本类型）
```
Class name: void
Simple name: void
is interface: false
is enum: false
is array: false
is primitive: true
```
----
上面的例子，基本上提到了所有可能出现的情况，类、接口、数组、枚举、基本类型（void）

Class都可以将他们的信息打印出来

（有意思的是：在打印`void`的`primitive`属性时，显示为`true`，对这个感兴趣可以自己搜一下哈，在《Java编程思想》这本圣经级别的书中，它把`void`也规定为了基本类型）

## `Class`也可以创建`class`实例
```java
Class cls = String.class;
String s = (String) cls.newInstance();
```
以上这个方法相当于`new String()`，通过`newInstance()`方法可以创建实例，但是有一定的局限：**只能调用`public`的无参数构造方法，带参数的构造方法，或者非`public`的构造方法都无法通过Class.newInstance()被调用**

将 `forName` 与 `newlnstance` 配合起来使用， 可以根据存储在字符串中的类名创建一个对象 
```java
String s = "java.util.Random";
Object m = Class.forName(s).newlnstance();
```

## 动态加载

在`JVM`执行`Java`程序时，并不是一次性加载所有的类到内存，而是第一次要用到的时候才回去加载

使用这个特性我们可以在运行期根据条件来控制加载`class`。





> 参考资料
[廖雪峰官方网站](https://www.liaoxuefeng.com/wiki/1252599548343744/1255945147512512)
官方API
《Java核心技术卷一》