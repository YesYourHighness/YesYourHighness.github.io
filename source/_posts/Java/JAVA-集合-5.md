---
title: JAVA-集合-5
date: 2019-08-14 17:17:18
tags: 
- Java
- Java集合
categories: 
- 后台
- Java
---

<center>
引言：Set接口及其集合、可变参数
</center>

<!--more-->

---


# HashSet

特点:
- 继承set接口
    1. 没有重复元素
    2. 没有索引，也不能使用普通的for循环
- HashSet特点
    1. 不同步（多线程）
    2. 无序的集合
    3. 没有索引
    4. 底层是一个哈希表结构（查询速度非常快）

## 包路径
```java
java.util
```

## 构造函数
```java
public HashSet()
//构造一个新的空 set，其底层 HashMap 实例的默认初始容量是 16，加载因子是 0.75。
```
```java
Set<Integer> set = new HashSet<>();//构造一个HashSet集合
set.add(1);
set.add(12);
set.add(12);//填入两个12
set.add(3);
System.out.println(set);//[1, 3, 12]只有一个12
```

## 存储自定义类型的元素
对于自定义元素
```java
Set<father> set = new HashSet<>();//构造一个HashSet集合
father f1 = new father("一号小弟",15);
father f2 = new father("二号小弟",15);
father f3 = new father("一号小弟",15);
//有三个变量，其实只有两个人，对于f1和f3是同一个人
set.add(f1);
set.add(f2);
set.add(f3);
System.out.println(set);
//但是发现，三个人都存进去了，这显然不是我们想要的
```
我们必须重写`hashCode`和`equals`方法，来保证不重复
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    father father = (father) o;
    return age == father.age &&
            Objects.equals(name, father.name);
}
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```
再次运行，发现重复的人就被去掉了

# 哈希值

哈希值是一个**十进制的整数**，由系统随机给出

就是对象的地址值，是一个逻辑地址，是模拟出来的地址，不是数据实际存储的物理地址

在Obejct类中有一个方法可以返回对象的哈希值
```java
father f1 = new father();
int h1 = f1.hashCode();
System.out.println(h1);//961
```
```java
String s1 = "重地";
String s2 = "通话";
System.out.println(s1.hashCode());//1179395
System.out.println(s2.hashCode());//1179395
//通话和重地有一个巧合，哈希值相同
```

## 哈希表

HashSet集合存储数据的结构（哈希表）

把元素进行了分组，相同哈希值的元素分为一组，每一组都是一个链表，这些元素的哈希值相同

如果链表超过了8位，那么就会把链表转化为红黑树，提高查询速度


- jdk1.8之前：哈希表 = 数组+链表

- jdk1.8之后：

    哈希表 = 数组 +链表
    
    哈希表 = 数组 + 红黑树（提高查询的速度）
    
    哈希表的特点：速度快 

## set不重复原理

add方法会计算字符串的哈希值，得到哈希值会去寻找是否存在此哈希值。

如果没有就会存储到集合当中

如果有(哈希冲突)就会调用`equals`方法对这两个相同的字符进行比较，比较如果不同才会存储这个元素，挂在同一个位置

# LinkedHashSet
继承了HashSet集合，底层是一个哈希表(数组+链表/红黑树)+链表

多了一条链表（记录元素的顺序），可以保证元素有序

## 包路径
```
java.util
```

**HashSet无序,LinkedHashSet有序**

# 可变参数
JDK1.5之后，如果我们需要接受多个参数，并且多个参数的类型一致，我们就可以简化成如下的格式：
```java
public static void sum(int... a){//这样接收
}
```
```java
//调用
sum(1,3,5,545,8,4);//可以接收任意个相同类型的参数
```
注意事项：
一个方法的参数列表，只能有一个 可变参数
如果方法的参数有多个，那么可变参数必须写在最右边

小实例：计算任意项的和
```java
public static void main(String[] args) throws ParseException {
    sum(1,3,5,545,8,4);
}
public static void sum(int... a){
    int sum = 0;
    for (int i : a) {
        sum+=i;
    }
    System.out.println(sum);//566
}
```