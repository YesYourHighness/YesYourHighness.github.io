---
title: JAVA-集合-1
date: 2019-08-14 17:17:04
tags: 
- Java
- Java集合
categories: 
- 后台
- Java
---

<center>
引言：Collection接口
</center>
<!--more-->

---

# 集合
Java中实现了部分的数据结构（Data Structure）的内容，并把他们封装成了类

## 集合是什么
集合是一种可以存储多个数据的容器（此集合非彼集合）

优点：
- 集合的长度可以随意变化
- 集合存储的都是对象，而且对象的类型可以不同

## 将集合接口与实现分离

例如：队列
> 队列：先进先出(FIFO)，头的一端可以删除元素，尾的一端可以添加元素，并且可以查找队列中元素的个数

队列Queue接口可能是这么实现的
```java
public interface Queue<E>{
    void add(E element);
    E remove();
    int size();
}
//这里只是做个示例
```
队列有两种实现：
1. 循环数组
```java
public class CircularArrayQueue<E> implements Queue<E>{
    private int head;
    private int tail;
    CircularArrayQueue(int capcity){...}
    public void add(E e){...}
    public E remove(){...}
    public int size(){...}
    private E[] elements;
}
//这里只是做个示例
```
2. 链表
```java
public class LinkedListQueue<E> implements Queue<E> {
    private Link head;
    private Link tail;
    LinkedListQueue(){...}
    public void add(E element){...} 
    public E remove(){...}
    public int size(){...}
}
//这里只是做个示例
```

当在程序中使用队列时，一旦构建了集合就不需要知道究竟使用了哪种实现，**所以，只有在构建集合对象的时候，使用具体的类才有意义**
，例如：
```java
Queue<MyClass> myClass = new CircularArrayQueue<>(100);
myClass.add(new MyClass("Jack"));
/*
    一旦我们改变了想法，我们只需要改变调用构造方法的那一块，如下
*/
Queue<MyClass> myClass = new LinkedListQueue<>(100);
myClass.add(new MyClass("Jack"));
```



## 集合关系图
在Java中，集合类的基本接口是Collection接口
![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic1/Collection01.png)

- Collection接口

    定义了所有单列集合中共性方法，
    所有单列集合都可以使用的方法，没有带索引的方法
    - List接口
        1. 有序的集合（存储和取出元素的顺序相同）
        2. 允许存储重复的元素
        3. 有索引，可以使用普通的for循环遍历
    - Set接口
        1. 不允许存储重复元素
        2. 没有索引，不能使用普通的for循环遍历
        3. 无序
    - LinkedHashSet
      
        例外，是有序的集合

学习集合的方法：
1. 学习顶层：学习顶层接口/抽象类的共性方法，所有的子类都可以使用
2. 使用底层：顶层无法创建对象使用，需要使用底层来创建对象

# Collection接口
```java
public interface Collection<E>{
    boolean add(E element);
    Iterator<E> iterator();
    ...
}
```
`iterator()`实现了Iterator接口，首先了解一下迭代器

## 迭代器

Iterator接口
```java
public interface Iterator<E>{
    E next();
    boolean hasNext();
    void remove();
    default void forEachRemaining(Consumer<? super E>action)
}
/*
Iterator迭代器接口有四个方法：
    1.  next()
    逐个访问集合中的每个元素，但是如果到带了集合的末尾会抛出一个NoSuchElementException异常
    Java的迭代器可以看做是位于两个元素之间的，当调用`next()`的方法时，迭代器就越过下一个元素，并返回刚刚越过的元素的引用
    2. hasNext()
    判断是否还有下一个元素
    3. remove()
    删除一个刚刚越过的元素
    4. forEachRemaining
    下面会说到
*/
```
![image](https://github.com/YesYourHighness/MyPicStore/raw/master/pic2/iterator_01.png)

### 迭代器的使用
迭代器的使用
```java
Collection<String> coll = ...;
Iterator<String> iter = c.iterator();
//调用集合的iterator()方法
while(iter.hasNext()){
    String e = iter.next();
    //do something
}
```
`for each`也可以简单的表示同样的循环操作
```java
for(String e : c){
    //do something
}
```
JavaSE8后，可以调用Iterator最后一个方法`foreachRemaining`方法，这个方法会提供一个Lambda表达式
```java
iter.forEachRemaining(element -> doSomething)
```



## Collection的常用方法
常用方法
```java
int size() 
//返回当前存储在集合中的元素个数

boolean isEmpty()
//如果此 collection 不包含元素，则返回 true。

boolean contains(Object o)
//判断当前集合是否包含给定的对象，包含返回true，不包含返回false

boolean containsAll(Collection<?> other)
//如果这个集合包含other中所有的元素，返回true

boolean add(E e)
/*如果此 collection 由于调用而发生更改，则返回 true。
如果此 collection 不允许有重复元素，并且已经包含了指定的元素，则返回 false。
*/
boolean addAll(Collection<? extends E> other)
//将other中所有的元素添加到这个集合

boolean remove(Object o)
//从此 collection 中移除指定元素的单个实例,不存在此元素返回false

boolean removeAll(Collection<?> other)
//从这个集合中删除other集合中存在的所有元素

boolean retainAll(Collection<?> other) 
//从这个集合中删除所有与 other 集合中的元素不同的元素。如果由于这个调用改变了集合， 返回 true

void clear()
//移除此 collection 中的所有元素,集合是还存在的

Object[] toArray()
//返回包含此 collection 中所有元素的数组。
```
## 代码示例
```java
Collection<String> coll = new ArrayList<>();//接口的构造
//add方法
boolean b1 = coll.add("张三");
//把给定的对象添加到集合当中，返回值一般都是true，可以不用接受
System.out.println(coll);//[张三] 不是地址,说明重写了toString方法
System.out.println(b1);//true

Collection<String> coll = new ArrayList<>();//接口的构造

coll.remove("张三");//删除方法

coll.contains("张三");//是否含有方法

coll.isEmpty(); //是否为空方法

coll.size();//获取长度

coll.toArray();//把集合变为数组

coll.clear();//清空元素方法
```

