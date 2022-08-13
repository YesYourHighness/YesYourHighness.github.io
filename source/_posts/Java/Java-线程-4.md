---
title: Java-线程-4
date: 2019-08-18 15:44:47
tags: 
- Java
- Java线程
categories: 
- 后台
- Java
---
<center>
引言：

函数式编程思想

Lambda表达式
</center>

<!--more-->

---

# 思想

- 面向对象的思想:

做一件事情，找一个能解决这个事情的对象，调用对象的方法，完成事情

- 函数式编程思想

能得到结果，怎么做的不重要


# 冗余的Runnable代码

我们学会了使用Runnable来创建多线程

## 正式方法

类
```java
public class RunnableImpl implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"新的线程创建了");
    }
}
```
主函数
```java
//创建接口的实现类
RunnableImpl run = new RunnableImpl();
//创建Thread类对象，构造方法中传递Runnable接口的实现类
Thread t = new Thread(run);
//调用start方法开启线程
t.start();
```

## 匿名内部类
```java
//使用匿名内部类实现多线程
Runnable r = new Runnable() {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "新的线程创建了");
    }
};
new Thread(r).start();
```
甚至更简单
```java
//甚至更简单
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "新的线程创建了");
    }
}).start();
```

## Lambda表达式
JDK1.8 引入了更加简便的方式
```java
new Thread(()->{
    System.out.println(Thread.currentThread().getName() + "新的线程创建了");
}).start();
```
由于只有一句话，所以可以省去括号和分号
```java
new Thread(()->
    System.out.println(Thread.currentThread().getName() + "新的线程创建了")
).start();
```

# Lambda表达式
## 标准格式
三部分组成:
1. 参数:多参数逗号分隔，无参数空下
2. 箭头
3. 一段代码
```java
(参数) -> {
    //代码
}
```
可以省略的内容
1. 参数列表：括号中的参数的数据类型可以不写
2. 参数列表：括号中的参数如果只有一个，那么类型和括号都可以省略
3. 一段代码：如果只有一行，无论是否有返回值，都可以省略{}，还可以省略return还有分号

注意：
省略必须一起省略

**使用前提：**

1. Lambda表达式必须有接口，且要求接口中有且只有一个抽象方法，无论是JDK内置的Runnable还是自定义接口，只有当接口中的抽象方法唯一时，才可以使用Lambda表达式
2. 使用Lambda必须要有上下文判断，也就是方法的参数或局部变量类型必须为Lambda对应的接口类型，才能使用Lambda作为接口的实例




