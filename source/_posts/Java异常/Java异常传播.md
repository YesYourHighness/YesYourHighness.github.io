---
title: Java异常传播
date: 2020-02-27 23:19:41
tags: 
- Java异常
categories: 
- Java
---


<center>
    引言：Java异常的传播
</center>

<!--more-->


# 异常的传播
本文不再讲解异常的基本知识点，进入更深层的异常学习，有关基础可以看[博客](https://yesyourhighness.github.io/2019/08/17/Java/Java-%E5%BC%82%E5%B8%B8/)


## 怎么看打印的异常信息

我们通常知道使用`e.printStackTrace()`来打印错误信息，但是怎么看呢？

示例:
```java
public class TempTest {
    public static void main(String[] args) {
        try {
            process1();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void process1() {
        process2();
    }

    static void process2() {
        Integer.parseInt(null); // 会抛出NumberFormatException
    }
}
```
运行后报出
```
java.lang.NumberFormatException: null
	at java.lang.Integer.parseInt(Integer.java:542)
	at java.lang.Integer.parseInt(Integer.java:615)
	at TempTest.process2(TempTest.java:20)
	at TempTest.process1(TempTest.java:16)
	at TempTest.main(TempTest.java:9)
//IDEA中可以点击括号内容对应找到原码
```
我们可以看出来，方法的运行顺序是
1. `main`调用了`process1`
2. `process1`调用了`process2`
3. `process2`调用了`Integer.parseInt(String s)`
4. `Integer.parseInt(String s`调用了`public static int parseInt(String s, int radix)`

打印结果如上，说明打印的结果是从最底层向最上层，层层打印出来的

点击第一行括号内容，就可以找到抛出异常的原码位置
```
public static int parseInt(String s, int radix) throws NumberFormatException {
    if (s == null) {
        throw new NumberFormatException("null");
    }
    ...
}
```
## 异常转换
观察以下代码
```java
public class TempTest {
    public static void main(String[] args) {
        try {
            process1();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void process1() {
        try {
            process2();
        } catch (NullPointerException e) {
            throw new IllegalArgumentException();
        }
    }

    static void process2() {
        throw new NullPointerException();
    }
}

```
分析一下：
1. `main`调用`process2`
2. `process2`抛出`NullPointerException`
3. `catch`捕获异常
4. `catch`抛出`IllegalArgumentException`

运行如下
```
java.lang.IllegalArgumentException
	at TempTest.process1(TempTest.java:19)
	at TempTest.main(TempTest.java:9)
```
发现，打印结果中只有`IllegalArgumentException`，看不到`NullPointerException`

我们发现异常丢失了原始的异常信息

怎么办呢？
```java
public class TempTest {
    public static void main(String[] args) {
        try {
            process1();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void process1() {
        try {
            process2();
        } catch (NullPointerException e) {
            throw new IllegalArgumentException(e);
            //改动在这里，将e作为了实例穿了进去
        }
    }

    static void process2() {
        throw new NullPointerException();
    }
}
```
再看打印信息
```
java.lang.IllegalArgumentException: java.lang.NullPointerException
	at TempTest.process1(TempTest.java:19)
	at TempTest.main(TempTest.java:9)
Caused by: java.lang.NullPointerException
	at TempTest.process2(TempTest.java:24)
	at TempTest.process1(TempTest.java:17)
	... 1 more
```
注意到`Caused by: Xxx`，说明捕获的`IllegalArgumentException`并不是造成问题的根源，根源在于`NullPointerException`，是在`TempTest.process2()`方法抛出的。


## 在Catch抛出异常

如果我们在`try`或者`catch`语句块中抛出异常，`finally`语句是否会执行？例如：

```java
public class Main {
    public static void main(String[] args) {
        try {
            Integer.parseInt("abc");
        } catch (Exception e) {
            System.out.println("catched");
            throw new RuntimeException(e);
        } finally {
            System.out.println("finally");
        }
    }
}
```

结果如下：
```
catched
finally
Exception in thread "main" java.lang.RuntimeException: java.lang.NumberFormatException: For input string: "abc"
    at Main.main(Main.java:8)
Caused by: java.lang.NumberFormatException: For input string: "abc"
    at ...
```
发现执行顺序如下;
1. 先执行`catch`
2. 再执行`finally`
3. 最后抛出`catch`中的异常

**因此，在catch中抛出异常，不会影响finally的执行。JVM会先执行finally，然后抛出异常。**

---
## 在finally抛出异常

如果在执行finally语句时抛出异常，那么，`catch`语句的异常还能否继续抛出？例如
```java
public class Main {
    public static void main(String[] args) {
        try {
            Integer.parseInt("abc");
        } catch (Exception e) {
            System.out.println("catched");
            throw new RuntimeException(e);
        } finally {
            System.out.println("finally");
            throw new IllegalArgumentException();
        }
    }
}
```
结果如下：
```java
catched
finally
Exception in thread "main" java.lang.IllegalArgumentException
    at Main.main(Main.java:11)
```
发现执行顺序如下;
1. 先执行`catch`
2. 再执行`finally`
3. 最后抛出`finally`中的异常

`catch`中抛出的异常去哪儿了？？

这个异常被**屏蔽**了（Suppressed Exception）

为了让他显示出来，我们可以使用一个实例将异常存起来，最后调用`addSuppressed()`方法
```java
public class Main {
    public static void main(String[] args) throws Exception {
        Exception origin = null;
        try {
            System.out.println(Integer.parseInt("abc"));
        } catch (Exception e) {
            origin = e;
            throw e;
        } finally {
            Exception e = new IllegalArgumentException();
            if (origin != null) {
                e.addSuppressed(origin);
            }
            throw e;
        }
    }
}

```
结果
```java
Exception in thread "main" java.lang.IllegalArgumentException
    at Main.main(Main.java:11)
Suppressed: java.lang.NumberFormatException: For input string: "abc"
    at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
    at java.base/java.lang.Integer.parseInt(Integer.java:652)
    at java.base/java.lang.Integer.parseInt(Integer.java:770)
    at Main.main(Main.java:6)
```
发现`Suppressed:`后面包含了被屏蔽的异常

绝大多数情况下，在`finally`中不要抛出异常。因此，我们通常不需要关心`Suppressed Exception`。



> 参考资料：[廖雪峰官方网站](https://www.liaoxuefeng.com/wiki/1252599548343744/1264738764506656)