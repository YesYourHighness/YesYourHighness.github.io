---
title: JAVA-集合-4
date: 2019-08-14 17:17:15
tags: 
- Java
- Java集合
categories: 
- 后台
- Java
---

<center>
引言：

List接口及其子类
</center>

<!--more-->

---

# List集合

## 特点

- 有序：存储与取出的顺序是一致的
- 有索引：包含常用的索引方法
- 允许重复

## 常用方法
```java
public void add(int index,E element)
//将指定的元素添加到指定的位置上
public E remove (int index)
//移除表中指定位置的元素，返回被移除的元素
public E set(int index,E element)
//用指定元素替换几何中指定位置的元素，返回更新前的元素
```
示例
```java
List<String>list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
list.add("d");

list.add(3,"啦啦啦");// 添加方法
System.out.println(list);
list.remove("啦啦啦");// 移除方法
System.out.println(list);
list.set(2,"A");// 替换方法
System.out.println(list);
```
遍历方法
```java
//普通for循环
for(int i = 0;i<list.size();i++){
    String s = list.get(i);
    System.out.println(s);
}
//foreach遍历
for (String s : list) {
    System.out.println(s);
}
//迭代器遍历
Iterator<String> it = list.iterator();
while (it.hasNext()){
    String s = it.next();
    System.out.println(s);
}
```
## 常见报错
```java
IndexOutofBoundsException//索引越界异常，集合会报
ArrayIndexOutofBoundsException//数组索引越界异常
StringIndexOutOfBoundsException//字符串索引越界异常
```

# ArrayList<E>

List 接口的大小可变数组的实现。

实现了所有可选列表操作，**并允许包括 `null`在内的所有元素。**

除了实现 List接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小

尖括号内的E叫做**泛型**，装在该集合当中的所有元素都必须是统一的类型

泛型**只能是引用类型，不能是基本类型**

包路径：
```
java.util
```

构造方法：
```java
public ArrayList(Collection<? extends E> c)
//构造一个包含指定 collection 的元素的列表，
//这些元素是按照该 collection的迭代器返回它们的顺序排列的。 
```
常用方法
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

示例代码：
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

# LinkedList

底层是一个双向列表，

查询慢，增删快，多线程

也有`list`的三个特点(有序，可索引，可重复)

## 包路径
```
java.util
```
## 常用方法

添加方法
```java
public void addFirst(E e)
//将指定元素插入此列表的开头

public void addLast(E e)
//将指定元素添加到此列表的结尾

public void push(E e)
//将元素推入此列表所表示的堆栈。换句话说，将该元素插入此列表的开头,同addFirst
```
获取方法
```java
public E getFirst()
//返回此列表的第一个元素

public E getLast()
返回此列表的最后一个元素
```

移除方法
```java
public E removeFirst()
//移除并返回此列表的第一个元素。 

public E removeLast()
//移除并返回此列表的最后一个元素。 

public E pop()
//从此列表所表示的堆栈处弹出一个元素。换句话说，移除并返回此列表的第一个元素。 
//此方法等效于 removeFirst()。 

```

## 示例代码
```java
LinkedList<String> linked = new LinkedList<>();
linked.add("a");
linked.add("b");
linked.add("c");
//向集合添加一些内容

//添加内容
linked.addFirst("www");
linked.push("aaa");//与addFirst相同
linked.addLast("com");
System.out.println(linked);
//获取内容
System.out.println(linked.getFirst());
System.out.println(linked.getLast());
//移除方法
linked.removeFirst();
linked.removeLast();
System.out.println(linked);
```

# Vector
是一个同步的集合（单线程，速度慢）

了解即可，最早期的一个集合