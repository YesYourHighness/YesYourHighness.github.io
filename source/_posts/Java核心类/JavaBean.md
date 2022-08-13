---
title: JavaBean
date: 2020-02-25 23:51:26
tags:
- Java核心类
- JavaBean
categories:
- Java核心类
- JavaBean
---
<center>
引言：JavaBean
</center>

<!--more-->

# JavaBean
来看一个小猫类
```java
public class Cat {
    private int age;
    private String name;
    private boolean health;

    public Cat() {
    }

    public boolean isHealth() {
        return health;
    }

    public void setHealth(boolean health) {
        this.health = health;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```
在编程过程中，我们发现很多类都和这个类有着很多共同点
- 实例字段用`private`修饰
- 实例方法用`public`修饰并且方法名是`getXxx()setXxx()`
这种类我们都叫**JavaBean**


## 属性
> 我们通常把一组对应的读方法（getter）和写方法（setter）称为属性（property）

> 属性只需要定义getter和setter方法，不一定需要对应的字段

```java
public class Cat {
    private int age;
    private String name;
    private boolean health;
    
    //没有字段但是也是属性
    public boolean isChild() {
        return age <= 6;
    }
    
    ...
    
}
```
## JavaBean的作用
JavaBean主要用来传递数据，即把一组数据组合成一个JavaBean便于传输。

此外，JavaBean可以方便地被IDE工具分析，生成读写属性的代码，主要用在图形界面的可视化设计中。


## 遍历JavaBean属性
要枚举一个JavaBean的所有属性，可以直接使用Java核心库提供的`Introspector`：

- Introspector:
> The Introspector class provides a standard way for tools to learn about the properties, events, and methods supported by a target Java Bean.
- BeanInfo
> Use the BeanInfo interface to create a BeanInfo class and provide explicit information about the methods, properties, events, and other features of your beans.

示例：打印小猫类的属性
```java
BeanInfo info = Introspector.getBeanInfo(Cat.class);
for(PropertyDescriptor pd : info.getPropertyDescriptors()){
    System.out.println(pd.getName());
    System.out.println(pd.getReadMethod());
    System.out.println(pd.getWriteMethod());
}
```
打印结果
```
age
public int Cat.getAge()
public void Cat.setAge(int)
child/*代码没有定义字段，但是却有这个字段，这个字段是被isChild方法带来的*/
public boolean Cat.isChild()
null
class/*class类是继承自Object类的属性getClass方法带来的*/
public final native java.lang.Class java.lang.Object.getClass()
null
health
public boolean Cat.isHealth()
public void Cat.setHealth(boolean)
name
public java.lang.String Cat.getName()
public void Cat.setName(java.lang.String)
```





> 知识来源：
[廖雪峰博客](https://www.liaoxuefeng.com/wiki/1252599548343744/1271993169413952)
及 官方api文档