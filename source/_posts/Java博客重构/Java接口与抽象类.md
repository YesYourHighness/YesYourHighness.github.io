---
title: Java接口与抽象类
date: 2019-08-10 10:51:21
tags: 
- Java
categories: 
- 后台
- Java
---
<center>
引言：接口与抽象类
</center>

<!--more-->

------

# 接口
接口是一种引用的数据类型，最重要的内容就是其中的抽象方法

创建格式
```java
public interface api {
}
```
换了关键字`interface`之后，编译生成的**字节码文件仍然是`.java -> .class`**

- Java 7开始，接口中可以包含的内容有：

    - **常量**

    - 抽象方法
- Java 8 开始，那么可以包含的还有：

    - **默认方法**（重点：为什么要引入默认方法？）

    - **静态方法**
- Java 9 开始：还能有
    - **私有方法**

## 抽象方法

**注意**：
1. 接口不能直接使用，必须有一个**实现类**来实现该接口
2. 接口的实现类必须**覆盖重写**接口的**所有抽象方法**
3. 实现类一般起名字在最后加上`Impl`
4. 接口的抽象方法的关键字必须是**固定的两个`public void`，但是他们可以省略**
```java
public interface api {
    public abstract void method();
    void method2(); // 这两个方法的定义效果一样
}
```
接口和实现类继承时使用`implements`实现
```java
public interface api {
    public abstract void method();
}
//接口
```
```java
public class apiImpl implements api {
    @Override
    public void method(){
        System.out.println("实现类的使用");
    }
}
//实现类
```
## 默认方法
Java 8开始就可以在接口中定义**默认方法**

> **目的是为了解决接口升级的问题**，因为在投入使用后，如果给接口新加抽象方法会报错，这时候可以使用默认的接口方法

**默认方法的特点就是，可以重写，也可以不重写**

```java
public interface api {
    public abstract void method();
    public default void method2(){
        // 有自己的实现题
        System.out.println("添加的默认方法");
    }
}
```
```java
public class apiImpl implements api {
    @Override
    public void method(){
        System.out.println("实现类的使用");
    }
    @Override
    public void method2(){
        System.out.println("默认的方法也可以进行重写");
    }
}
```
## 静态方法
Java 8开始：同样，**接口的静态方法也是属于这个接口的，调用时直接使用接口名调用即可**

```java
public interface api {
    public static void staticMethod(){
        System.out.println("静态方法");
    }
}
```
```java
public static void main(String[] args) {
    api.staticMethod();
    //直接使用接口名调用即可
}
```
## 私有方法
Java 9出现的接口的私有方法

目的：是**为了简化接口中默认方法的重复部分**

```java
public interface api {
    public default void method1(){
        intro();
    }
    public default void method2(){
        intro();
    }
    private void intro(){
        System.out.println("你好");
    }
}
```
## 常量

接口中可以定义成员变量，**但是必须使用`public static final`三个修饰符来修饰（可以省略）**
```java
public static final int NUM = 10;
// 前三个关键字可以省略，接口常量一旦赋值便不可更改
```
注意：常量一般要完全大写，而且多单词要用下划线进行分隔

同样常量属于接口，可以直接用接口名调用
```java
public static void main(String[] args) {
    System.out.println(api.NUM);
    //调用接口常量直接使用接口名
}
```

## 接口与类的区别

实现接口与实现类的区别：
1. 接口**没有静态代码块**、接口**没有构造方法**
3. 一个类的**直接父类是唯一**的，但是一个类**可以实现多个直接接口**

```java
public class MyInterfaceImpl implements MyinerfaceA,MyinterfaceB{
//覆盖重写所有的抽象方法
}
```
4. 如果实现类实现的**多个接口存在方法同名的情况，只需要覆盖重写一次即可**
5. 如果没有覆盖重写所有的抽象方法，那么本类也只能是一个抽象类
6. 如果多个接口**存在重复的默认方法，实现类也要对其覆盖重写**
7. 一个类如果**直接父类和接口当中的方法产生了冲突，会优先使用父类当中的方法**

## 接口继承接口
1. **接口之间可以多继承**
2. 多个父接口抽象方法同名没有关系
3. 多个父接口的**默认方法同名必须进行默认方法的覆盖重写**

# 抽象类

定义：加关键字`abstract`

```java
public abstract class dog {
    public abstract void eat();//抽象方法
}//抽象类
```

注意：

1. 抽象方法没有实现体
2. **只有抽象类才能拥有抽象方法**
3. **不能直接`new`抽象对象**
4. 必须用子类来继承抽象父类
5. 子类**必须覆盖重写**抽象父类的**所有**的抽象方法，**如果没有实现完所有的方法，那么这个子类还是一个抽象类**
6. 抽象类不能创建对象
7. **抽象类可以有构造方法，供子类创建对象时初始化父类对象**

```java
public abstract class father {
    father(){
        System.out.println("父类抽象函数的构造");
    }
}
```

```java
public class son extends father {
    son(){
        System.out.println("子类构造函数");
    }
}
```

```java
public static void main(String[] args) {
   son xxx = new son();
}
//打印出
//父类抽象函数的构造
//子类构造函数
```

8. **抽象类不一定含有抽象方法，但是含有抽象方法的一定是抽象类**
