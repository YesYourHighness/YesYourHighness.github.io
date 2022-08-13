---
title: JAVA-集合-2
date: 2019-08-14 17:17:08
tags: 
- Java
- Java集合
categories: 
- 后台
- Java
---

<center>
引言：Iterator接口、foreach循环、泛型
</center>

<!--more-->

---

# Iterator

迭代器： 一种通用的取出集合元素的方法

`Iterator`是一个接口，我们无法直接使用

需要使用实现类对象，获取它的实现类比较特殊

迭代： 在取元素之前判断集合中有没有该元素，如果有就取出这个元素，继续判断，直到取出集合中所有需要的元素

## 实现类

获取实现类的方法

Collection接口有一个`iterator()`方法，此方法直接返回实现类对象

## 常用方法

```java
boolean hasNext()
//如果仍有元素可以迭代，则返回 true。（换句话说，如果 next 返回了元素而不是抛出异常，则返回 true）。 

E next()
//返回迭代的下一个元素。 
```

## 常用方法

```java
Collection<String> coll = new ArrayList<>();
coll.add("姚明");
coll.add("科比");
coll.add("麦迪");
coll.add("詹姆斯");
coll.add("艾弗森");
//iterator<>也有泛型
Iterator<String> it = coll.iterator();//Iterator有泛型<>
//通过Collection的Iterator()方法获得实现类
while (it.hasNext())//判断是否含有下个元素
    System.out.println(it.next());//循环获得元素
```

# foreach

JDK1.5之后的一个高级for循环，专门用来遍历数组和集合的，它的内部其实是个iterator迭代器

**所以在遍历的过程中，不能对集合中的元素进行增删操作**

所有的单列集合都可以使用增强for


## 格式
```java
for(元素的数据类型 变量 : Collection集合/数组){
    //操作代码，不能对集合中的元素进行增删操作
}
```

## 示例

```java
Collection<String> coll = new ArrayList<>();
coll.add("姚明");
coll.add("科比");
coll.add("麦迪");
coll.add("詹姆斯");
coll.add("艾弗森");

for (String s : coll) {
    System.out.println(s);
}
```

# 泛型

泛型是一种未知的数据类型，当我们不知道使用声明数据类型的时候，我们可以使用泛型

可以看成一个变量，可以用来接收数据类型

`ArrayList`集合在定义的时候，不知道集合中都会存储声明元素的数据，所以类型使用泛型

```java
public class ArrayList<E>{
    public boolean add(E e){}
    public E get(int index){}
}
```
泛型数据类型的确定时间在创建对象的那一刻

## 优缺点

- 优点
    1. 避免了类型转换的麻烦，存储的是什么类型，取出的就是什么类型
    2. 把运行期异常(代码运行之后会抛出的异常)，提升到了编译器(写代码的时候会报错)

- 弊端
  
    泛型是什么类型只能存储声明类型

如果不加泛型，默认`Object`类，可以存储任何对象，**但不安全，但容易引发异常**

（例如：当存入`Integer`对象和`String`对象，遍历时想调用`toString`方法，而这样就必须向下转型，在运行中也可能会报错）

## 泛型类

### 泛型类
```java
public class father<E> {
    private E name;

    public void setName(E name) {
        this.name = name;
    }

    public E getName() {
        return name;
    }
}
```

调用
```java
father f = new father();
f.setName("字符串");
f.setName(123);
f.setName(true);
//不管是什么类型都可以使用
```

### 泛型方法
```java
public <E> void method (E e){
    System.out.println(e);
}
public static <S> void methodStatic(S s){
    System.out.println(s);
}
```

### 泛型接口

```java
public interface api<E> {
    public void out(E e);
}
```

接口实现类
```java
public class apiImpl <E> implements api<E>{
    @Override
    public void  out(E e){
        System.out.println(e);
    }
}
```

### 高级用法

泛型通配符？

不知道使用什么类型时可以使用

但是此时只能接受数据，而不能存储数据
```java
public static void print(ArrayList<?> list){
    Iterator<?> it = list.iterator();
    while (it.hasNext()){
        System.out.println(it.next());
    }
}
```

高级用法,看懂即可

泛型的上限限定：`? extends E` 代表使用的泛型只能是E类型的子类/本身

泛型的下限限定：`?  super  E`		代表使用的泛型只能是E类型的父类/本身