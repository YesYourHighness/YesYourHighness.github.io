---
title: Java-异常
date: 2019-08-17 21:39:11
tags: 
- Java
- Java异常
categories: 
- 后台
- Java
---

<center>
引言：

异常是什么？

怎么处理异常？
</center>

<!--more-->

---

# 异常

在程序执行的过程中，出现的非正常的情况，导致JVM非正常停止

异常本身是一个类，产生异常就是创建异常对象并抛出了一个异常对象，java处理的方式是中断处理。

异常不是语法错误！！

异常体系：

异常的最顶层的父类`java.lang.Throwable`

子类：`java.lang.Error`与`java.lang.Exception`


```
graph LR
Error-->Throwable
Exception-->Throwable
RuntimeException-->Exception
```
## 处理
第一种：抛出，交给虚拟机处理

虚拟机的处理方式：中断处理，把异常打印在控制台上
```java
public static void main(String[] args) throws ParseException {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    Date date = sdf.parse("1999-09-09");
}
```
第二种：try/catch
优点：程序抛出错误后，还可以继续运行。

Error的问题必须修改代码

## 异常产生的过程

第一步：

在遇到不正常的情况时，jvm会做两件事情
1. 根据异常产生的原因创建一个异常对象，这个对象包含了一场产生的内容，原因，位置
2. 在出现异常的方法中，没有异常的处理逻辑（没有`try/catch`），那么jvm就会把异常调出给方法的调用者，main方法来处理这个异常

第二步：

如果main无法处理这个异常，则会把异常抛出给main方法的调用者——jvm；

第三步：

JVM接受到这个异常，继续做两件事情
1. 打印异常
2. JVM中止当前的java程序——中断处理

## 异常的处理
五个关键字：
```java
try catch finally throw	throws
```

### throw
可以抛出指定的异常
`throw new xxException("异常产生原因")`
注意：
1. `throw`关键字必须写在方法的内部
2. `throw`关键字后边`new`的对象必须是`Exception`或者是`Exception`的子类对象
3. `throw`关键字抛出指定的异常对象，我们就必须处理这个异常对象
    1. `throw`关键字后边创建的如果是`RuntimeException`或者是`RuntimeException`的子类对象，我们可以不处理，默认交给jvm处理（打印异常对象，中断程序）
    2. `throw`关键字后边创建的如果是编译异常，我们必须处理这个异常，要么`throws`要么`try/catch`
4. 空指针异常`NullPointexception`是一个运行期异常，我们不必处理
5. `ArrayIndexOutOfBoundsException`也是一个运行期异常

注意：

工作中我们要对传递过来的参数进行合法性校验，

如果参数不合适，那么我们就必须使用异常给定方式，告知方法的调用者，传递的参数有问题。
```java
    private static void get(int[]arr,int index){
        if(arr==null)
            throw new NullPointerException("传递的数组的值是空");
        if(index<0||index>arr.length-1)
            throw new ArrayIndexOutOfBoundsException("超出范围");
        System.out.println(arr[index]);
    }
```
### throws
异常处理的第一种方式：交给别人处理

作用：

当方法内部抛出异常对象的时候，我们就必须处理这个异常对象，

可以使用throws关键字处理异常对象，会把异常对象声明抛出给方法的调用者处理（自己不处理，给别人处理），最终交给JVM处理——中断处理。

```java
//方法声明时使用
修饰符  返回值类型  方法名（参数）throws{
throw new xxxException("产生原因");
...
}
```
注意：
1. 必须写在方法的声明处
2. 声明的异常必须是`Exception`或者他的子类
3. 方法内部如果抛出了多个异常对象，那么`throws`后边也必须声明多个异常
4. 如果抛出的异常对象有子父类关系，则只声明父类异常即可
5. 调用了一个声明抛出异常的方法，我们就必须的处理声明的异常，要么继续使用`throws`声明抛出，交给方法的调用者处理，最终交给`JVM`，要么`try/catch`自己处理异常
6. `FileNotFoundException`是编译异常，抛出了编译异常就必须处理这个异常
7. `IOException`也是一个编译异常，必须处理
8. 中断交给异常处理之后不会再执行之后的代码

```java
public static void main(String[] args) throws IOException {
    readFile("d:\\a.txt");
}
public static void readFile(String filename) throws IOException{
    if(!filename.endsWith(".txt")){
        throw new IOException("文件名后缀不正确");
    }
}
```
### try/catch
第二种处理方式：自己处理异常
```java
try{
//可能产生异常的代码
}catch（
//这里定义一个异常的变量，用来接收try中抛出的异常对象）{
//异常的处理逻辑，产生异常对象之后，怎么处理异常对象
//一般在工作中会把异常的信息记录到一个日志中
}...catch可以有多个
```
注意：
1. try中可能会抛出多个异常对象，那么就可以使用多个catch来处理这些异常对象
2. 如果try中产生了异常，那么就会执行catch中的异常处理逻辑，执行完毕之后，继续向下执行
3. 如果try中没有产生异常，则catch中的代码不会执行，将继续执行之后的代码
4. catch里面定义的异常变量，如果有子父类关系，那么子类的异常变量必须在上

```java
public static void main(String[] args)  {
    try{
        //try中放有异常的代码
        readFile("d:\\a.tx");
    }catch(IOException e){
        //try中抛出什么异常对象，catch就定义什么异常变量用来接收这个异常对象
        //这里放异常的处理逻辑
        System.out.println("传递的文件的后缀不是txt");
    }
    System.out.println("后续会继续执行");
}
public static void readFile(String filename) throws IOException{
    if(!filename.endsWith(".txt")){
        throw new IOException("文件名后缀不正确");
    }
}
```
### finally
finally代码块：无论是否出现异常，都要执行

注意事项：

1. 不能单独使用
2. finally一般用于资源释放（资源回收），无论程序是否出现异常，最后都要资源释放
```java
 try{
    readFile("d:\\a.tx");
}catch(IOException e){
    System.out.println("传递的文件的后缀不是txt");
}finally {
    System.out.println("后续会继续执行");
}
```

# Throwable
Throwable 类是 Java 语言中所有错误或异常的超类

## 包路径
```
java.lang
```

## 常用方法
```java
public String getMessage()
//返回此 throwable 的详细消息字符串。

public String toString()
//返回此 throwable的简短描述。结果是以下字符串的串联： 
//此对象的类的 name ": "（冒号和一个空格） 调用此对象 getLocalizedMessage() 方法的结果 
//如果 getLocalizedMessage 返回 null，则只返回类名称。

public void printStackTrace()
//将此 throwable 及其追踪输出至标准错误流。

```
示例代码：
```java
public static void main(String[] args) {
    try {
        readFile("d:\\a.tx");
    } catch (IOException e) {
        System.out.println("传递的文件的后缀不是txt");
        System.out.println(e.getMessage());//文件名后缀不正确
        System.out.println(e.toString());//java.io.IOException: 文件名后缀不正确
        e.printStackTrace();
        //java.io.IOException: 文件名后缀不正确
        //	at cn.itcast.day04.demo01.demo.readFile(demo.java:22)
        //	at cn.itcast.day04.demo01.demo.main(demo.java:11)
    }
}
public static void readFile(String filename) throws IOException {
    if (!filename.endsWith(".txt")) {
        throw new IOException("文件名后缀不正确");
    }
}
```
# 自定义异常

Java自带的异常类不够我们使用，我们需要自定义一些异常类
```
public class xxxException extends Exception|RuntimeException{
    //添加一个空参构造
    //添加一个带异常信息的构造方法
}
```
注意：

1. 自定义异常类一般都是以`Exception`结尾，说明该类是一个异常类
2. 自定义异常类，必须得继承`Exception`或者`RuntimeException`
    1. `Exception`:那么自定义的异常类就是一个编译期异常，如果方法内部抛出了异常，就必须处理这个异常，要么`throws`要么`try/catch`
    2. `RuntimeException`：那么自定义的异常类就是一个运行期异常，无需处理，交给虚拟机

```java
public class RegisterException extends Exception{
    public RegisterException(){}
    //空参构造
    public RegisterException(String msg){
        super(msg);
    }
    //一个带异常信息的构造方法
}
```
所有的异常类都会有一个带异常信息的构造方法，

方法内部会调用父类带异常信息的构造方法，让父类来处理这个异常信息

父类错误，子类也会跟着报错，不论子类是否错误

