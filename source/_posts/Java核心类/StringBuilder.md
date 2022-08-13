---
title: StringBuilder
date: 2020-02-24 21:52:53
tags:
- Java核心类
- StringBuilder
categories:
- Java核心类
- StringBuilder
---

<center>

引言: StringBuilder

</center>

<!--more-->

# StringBuilder


## 为什么要有StringBuilder

我们先不学习StringBuilder，先来思考一个问题：为什么Java要引入这么一个类呢？

看以下代码
```java
String str = 1 + "2" + 3;
```
这一行代码，会产生 `"12"`、`"123"`两个字符串对象
，但是我们`"12"`其实并不是我们需要的，他是Java创建的临时对象，浪费内存，还会浪费GC(Java垃圾清理)效率

为了高效的提高直接拼接字符串的效率，所以java提供了StringBuilder这个类

如果我们使用`StringBuilder`这个类，在运行上面代码时，不会创建新的临时对象了

jdk api 官方说明：
> This class is designed for use as a drop-in replacement for StringBuffer in places where the string buffer was being used by a single thread (as is generally the case). Where possible, it is recommended that this class be used in preference to StringBuffer as it will be faster under most implementations.

## 构造方法

```java
StringBuilder()
//无参构造
StringBuilder(CharSequence seq)
//CharSequence是一个接口，String、StringBuilder、StringBuffer实现了这个接口，其实代表的就是一个字符串
StringBuilder(int capacity)
//capacity参数是缓存大小
StringBuilder(String str)
//通过一个字符串构造
```


## 常见方法
StringBuilder的常用方法主要有两个，他们都有多个重载方法，匹配了各种各样的数据类型
   1. apped
   2. insert
### apped
```java
StringBuilder append(String str)
//添加一个字符串到调用字符串的末尾
```
例如
```java
StringBuilder sb = new StringBuilder("你好");
System.out.println(sb.append("晚安"));//你好晚安
```
append方法可以链式调用
```
StringBuilder sb = new StringBuilder("你好");
sb.append("晚安")
        .append("去哪儿")
        .append(1);
System.out.println(sb);//你好晚安去哪儿1
```

#### 补充：链式调用
实现链式调用的秘诀在于返回自身，我们也可以实现一个可以链式调用的方法
```java
public class Chain {
    private int sum =0;

    public Chain add(int x){
        sum+=x;
        return this;
    }

    public int getSum() {
        return sum;
    }
}

public static void main(String[] args){
    Chain c = new Chain();
    c.add(5).add(6).add(8);
    System.out.println(c.getSum());
}
```



### insert
```java
StringBuilder insert(int offset, String x)
//offset 第几个开始插入,从1开始
//x 插入的内容
```
例子：
```java
StringBuilder sb = new StringBuilder("你好");
sb.insert(1,"晚安");
System.out.println(sb);//你晚安好
```
insert同样可以链式编程

值得注意的是，`sb.append(x)`和`sb.insert(str.length(), x)`其实作用是相同的
```java
StringBuilder sb1 = new StringBuilder("你好");
StringBuilder sb2 = new StringBuilder("你好");
sb1.insert("你好".length(),"晚安");
sb2.append("晚安");
System.out.printf("%s , %s",sb1,sb2);//你好晚安 , 你好晚安
```

### 其他方法
```java
StringBuilder delete(int start, int end)
//删除方法，从0开始，比如我想删除第一个字符，delete(0,1);  删除所有的delete(0,str.length())
int	capacity()
//返回当前StringBuilder的缓存容量
```


> 知识来源：
[廖雪峰博客](https://www.liaoxuefeng.com/wiki/1252599548343744/1271993169413952)
及 官方api文档