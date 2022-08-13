---
title: Java-API-1
date: 2019-08-11 22:03:34
tags: 
- Java
- JavaAPI
categories: 
- 后台
- Java
---
<center>
引言：

API是什么？

如何使用呢？

Scanner类

Random类

ArrayList类

</center>

<!--more-->

---

# API

## API是什么

Application Programming Interface

应用程序编程接口，其实就是一堆前辈已经写好的，可以直接复用的常用组件

## API文档的使用

第一看 包路径

第二看 构造方法

第三看 普通方法

## 引用API的使用步骤

1. 导包

    `import 包路径. 类名称`

    如果需要使用的目标类和当前类处于同一个包下，那么就不需要导包了
    
    只有`java.lang`包下的内容不需要导包，其他都需要用到`import`语句
2. 创建
    `类名称 对象名 = new 类名称();`
3. 使用 
    `对象名.成员方法名()`

-------

那么开始学习第一个API吧

----

# Scanner

一个可以使用正则表达式来解析基本类型和字符串的简单文本扫描器。

Scanner 使用分隔符模式将其输入分解为标记，默认情况下该分隔符模式与空白匹配。然后可以使用不同的 next 方法将得到的标记转换为不同类型的值。 

## 包路径：
```
java.util
```

## 构造方法:
```java
//有很多构造方法，包括从文件中读入的scanner构造
//目前只需要了解以下即可
public Scanner(InputStream source)
// 从指定的输入流扫描，并通过默认字符集转换为字符
```
## 常用方法：
```java
public boolean nectInt()    //输入int变量
public double nectDouble()  //输入double变量
public float nectFloat()    //输入float变量
public String nect()        //输入String变量
public byte nextByte()      //输入Byte变量
public short nectShort()    //输入short类型
public long nectInt()       //输入long类型
public boolean nextBoolean()//输入boolean类型
```

## 示例代码：
```java
Scanner sc = new Scanner(System.in);//构造一个系统输入的Scanner类
//调用最常见的方法
int x1 = sc.nextInt();
String x2 =sc.next();
long x3 = sc.nextLong();
double x4 = sc.nextDouble();
byte x5 = sc.nextByte();
short x6 = sc.nextShort();
boolean x7 = sc.nextBoolean();
float x8 = sc.nextFloat();
```

# Random

使用了48位的种子，使用线性同余公式生成**伪随机数流**

## 包路径：
```
java.util
```

## 构造方法：
```java
public Random()

// 创建一个新的随机数生成器。
// 此构造方法将随机数生成器的种子设置为某个值，
// 该值与此构造方法的所有其他调用所用的值完全不同。 
```
## 常用方法：
```java
public int nextInt()
//返回下一个伪随机数，它是此随机数生成器的序列中均匀分布的 int 值
public int nextInt(int n)
//返回一个伪随机数，它是从[0,n)的一个随机的整型，左开右闭区间
```
## 示例代码：
```java
Random r = new Random();// 构造Random类
int num = r.nextInt(5);//返回一个[0,5)的随机数
System.out.println(num);
int num2 = r.nextInt(5)+1;//返回一个[0,5]的随机数,只需要加一个1即可
```

# ArrayList<E>

List 接口的大小可变数组的实现。

实现了所有可选列表操作，并允许包括 null 在内的所有元素。

除了实现 List接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小

尖括号内的E叫做**泛型**，装在该集合当中的所有元素都必须是统一的类型

泛型**只能是引用类型，不能是基本类型**

## 包路径：
```
java.util
```

## 构造方法：
```java
public ArrayList(Collection<? extends E> c)
//构造一个包含指定 collection 的元素的列表，
//这些元素是按照该 collection的迭代器返回它们的顺序排列的。 
```
## 常用方法
```java
public E set(int index,E element);  //输入索引值和相同的泛型内容替换原有的内容

public E get(int index)             //返回索引值对应的内容

public boolean add(E e)             //把指定元素添加到集合的尾部

public void add(int index,E element)//添加指定的元素到索引值处，后面的内容索引值加一

public E remove(int index)          //移除集合上指定位置上的元素

public boolean remove(Object o)     
//移除此列表中首次出现的指定元素（如果存在）。如果列表不包含此元素，则列表不做改动

public void clear()                 //移除集合中的所有元素

public boolean addAll(Collection<? extends E> c)
//将另一个集合并入此集合

public boolean addAll(int index,Collection<? extends E> c)
//从指定位置并入另一个集合

public void ensureCapacity(int minCapacity)
//增加此ArrayList的容量,以确保至少能容纳参数所指定的元素数

public boolean isEmpty()            //判断集合是否为空

public boolean contains(Object o)   //判断集合是否含有指定的元素

public int indexOf(Object o)
//返回此列表中首次出现的指定元素的索引，或如果此列表不包含元素，则返回 -1。

public int lastIndexOf(Object o)
//返回此列表中最后一次出现的指定元素的索引，或如果此列表不包含索引，则返回 -1。

```

## 示例代码：
```java
ArrayList<String> list = new ArrayList<>();
// 创建了一个ArrayList的集合，
// 泛型为String类型，代表这个集合内只能含有String类型的数据
System.out.println(list);//返回[]
//直接打印空集合，返回的并不是地址，而是一个空数组

list.add("小明");//添加方法
list.add(1,"小白");//指定添加

String str = list.get(0);//得值方法

list.set(0, "小红");//替换方法

list.remove(0);//移除方法
list.remove("小白");//移除第一个指定值

list.ensureCapacity(9);//最小兼容方法

list.isEmpty();//是否空方法

list.contains("小白");//是否包含方法

list.indexOf("小白");//求首次出现索引方法

list.lastIndexOf("小白");//求末次出现索引方法


ArrayList<String> list2 = new ArrayList<>();
list2.addAll(list);//合并另一个ArrayList

list.size();//返回长度

for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}//输入list.fori+TAB快速得到遍历集合的循环
```

泛型只能导入引用类型，那怎么存入基本类型呢？

使用基本类型的对应的包装类

从JDK1.5开始，支持自动装箱拆箱
```java
    ArrayList<Integer> list1 = new ArrayList<>();
    list1.add(100);//完成了自动装箱
    list1.add(new Integer(100));//手动装箱，同上
```