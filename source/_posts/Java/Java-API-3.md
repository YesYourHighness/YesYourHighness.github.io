---
title: Java-API-3
date: 2019-08-12 20:53:34
tags: 
- Java
- JavaAPI
categories: 
- 后台
- Java
---
<center>
引言：

Math类

Object类

Objects类

</center>

<!--more-->

------

# Math

Math 类包含用于执行基本数学运算的方法，如初等指数、对数、平方根和三角函数。 

## 包路径
```
java.lang
```

## 构造方法

直接通过Math调用，没有构造函数

## 常用方法

```java
public static double abs(double num)//获取绝对值
public static double ceil(double num)//向上取整
public static double floor(double num)//向下取整
public static long round(double num)//四舍五入
Math.PI	//代表近似的圆周率常量
```

## 示例代码

```java
int x = -8;
double y = 2.6;
System.out.println(Math.abs(x));    // 8  绝对值
System.out.println(Math.ceil(y));   // 3.0 向上取整
System.out.println(Math.floor(y));  // 2.0 向下取整
System.out.println(Math.round(y));  // 3  四舍五入
System.out.println(Math.PI);        // 3.1415926 圆周率
```

# Object

类 Object 是类层次结构的根类。

每个类都使用 Object 作为超类。所有对象（包括数组）都实现这个类的方法。 

## 包路径
```
java.lang
```

## 构造方法
```java
public Object()
```

## 常用方法
```java
public String toString() 
// 返回该对象的字符串表示
//该字符串由类名（对象是该类的一个实例）、at 标记符“@”和此对象哈希码的无符号十六进制表示组成

public boolean equals(Object obj)
//
```

## 示例代码

toString
```java
father f = new father();
String str = f.toString();
System.out.println(str);//cn.itcast.day04.demo01.father@1b6d3586
System.out.println(f);//cn.itcast.day04.demo01.father@1b6d3586
// 直接打印一个对象直接调用了toString方法
```
但是直接打印一个对象的地址没有什么意义，我们重写它的String方法
```java
public class father {
    public String name;
    @Override
    public String toString(){
        return "Person:"+name;
    }//这时候再打印就会返回我们重写之后的形式
}
//看一个类是否重写了toString我们直接打印这个类的对象即可，
//如果没有重写toString方法，打印的就是对象的地址值
```

equals
```java
//源码：从源码可以看出，默认的比较是比较地址的位置
public boolean equals(Object obj){
return(this  ==  obj);
}
//原本的比较
father f1 = new father("A");
father f2 = new father("A");
System.out.println(f1.equals(f2));// false 比较地址，地址不同
f1 = f2;
System.out.println(f1.equals(f2));// true 地址相同
```
但是我们比较他们的地址没有意义，所以我们重写一下equals方法
```java
@Override
public boolean equals(Object obj){
    //因为obj是Object的子类，我们必须要向下转型,否则不能调用子类新增的成员
    father f = (father)obj;
    return this.name.equals(f.name);
}
```
又考虑到传入空或者传入其他类型的对象，可能会报错，我们完善一下重写的方法

使用ALT+insert即可
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;//地址相同直接返回true
    if (o == null || getClass() != o.getClass()) return false;
    // 如果是一个null返回false
    //getClass()!= o.getClass()利用反射技术，判断o是否是原本的类型
    // 等效于 o instanceof father 这句话
    father father = (father) o;
    return Objects.equals(name, father.name);
    //object的方法容易抛出空指针异常，Objects解决了这个问题
}

@Override
public int hashCode() {
    return Objects.hash(name);
}//对于集合的方法
```

# Objects
JDK7添加，提供了一些静态方法来操作对象，这些方法是空指针安全的

## 包路径
```
java.util
```

## 构造函数
无构造方法，直接调用Objects来使用

## 常用方法
```java
//equals源码
public static boolean equals(Object a,Object b) {
return  (a==b)  ||  (  a!=null && a.equals(b)  );
//先比较地址，再判断是不是空指针，然后再调用Object的
}

requireNonNull()//查看是否为空对象
//有两个参数，第一个是要判断的对象，第二个是放置想要报出的错误信息
```

## 示例代码

直接使用Obejct的equals会报空指针异常
```java
    String s1 = null;
    s1.equals('a');
    //报错Exception in thread "main" java.lang.NullPointerException
```
而Obejcts不会报错
```java
    String s1 = null;
    Objects.equals(s1, "a");
```

查看是否为空对象
```java
    String s1 = null;
    Objects.equals(s1, "a");
```

