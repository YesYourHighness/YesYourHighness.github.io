---
title: Java基础扩展
date: 2019-08-26 22:00:03
tags: 
- Java
categories: 
- 后台
- Java
---

<center>
引言：

衔接SE基础部分，加强基础

Junit

反射

注解
</center>

<!--more-->

-----

# Junit

 测试的一些分类：
1. 黑盒测试

    看不到代码什么样子，只需要给盒子输入一些参数，看看代码能不能输出预期的结果
    
    基本上很多测试人员都是黑盒测试的
    
2. 白盒测试

    白盒要关注进行的具体流程，需要关注代码


## Junit的使用

步骤：
1. 定义一个测试类（测试用例）
    - 类名：被测试的类名+Test
    - 包名：`xxx.xxx.xx.test`
2. 定义测试方法：可以独立运行
    - 方法名：test测试的方法名 (testAdd)
    - 返回值：建议使用void，无返回值
    - 参数列表：空参
3. 给方法加注解`@Test`
4. 导入Junit依赖环境(ALT+Enter导入包)

判定结果：

红色代表失败

绿色代表成功

```java
@Test
public void testAdd(){
    //此处创建对象，调用测试方法
    Calculator c = new Calculator();
    int result = c.add(1, 2);
    //System.out.println(result);
    //一般不会直接打印它的结果，而是做一个断言的操作
    //断言：我断言这个结果是3
    assert result==3?true:false;
}
```



## @Before和@After

- @Before

    修饰的方法会在测试方法之前被自动执行

- @After

    修饰的方法会在测试方法之后被自动执行

----
注意：
**Junit已经过时了解即可**

# 反射

框架设计的灵魂

框架：半成品软件，简化编码


## Java代码经历的三个阶段

- 第一阶段：**Source源代码阶段**
  
    `Person.java` --> javac编译 --> `Person.class`
    
    Person.class分为三层，成员变量，构造方法，成员方法

    代码还在硬盘上

- 第二阶段：**Class类对象阶段**

    类加载器，ClassLoader
    
    Class类对象，用来描述所有的源代码而文件的行为
   
    把第一阶段的三层分为三个对象 
```
成员变量-->Field对象数组
构造函数-->Constructor对象数组
成员方法-->Method对象数组
```


- 第三阶段：**Runtime运行时阶段**

    创建Person对象  new Person


## 反射的好处

> 反射： 将类的各个组成部分封装为其他对象，这就是反射机制

好处：
1. **可以在程序运行时，操作这些对象**
2. **可以解耦**（降低程序的耦合性），提高程序的可扩展性


## 反射获取字节码Class对象的三种方式

1. 获取成员变量
```java
Field[] getFields()//获取所有public修饰的成员变量
Field getField(String name)//获取指定名称public修饰的成员变量 

Field[] getDeclaredFields()//获取所有成员变量
Field getDeclaredFields(String name)

Field类1的方法：
//获取值
public Object get(Object obj)
//设置值
public void set(Object obj,Object value)
```

示例代码
```
Class personClass = Person.class;
//0.获取Person的Class对象
Field[] fields = personClass.getFields();
for (Field field : fields) {
    System.out.println(field);
}
Field a = personClass.getField("a");
//获取成员变量a的值
Person p = new Person();
a.set(p, "张三");
//Field的set方法
Object o = a.get(p);
//Field的get方法
System.out.println(o);
//打印出  张三


Field[] declaredFields = personClass.getDeclaredFields();
//获取所有的成员变量，不考虑修饰符
for (Field declaredField : declaredFields) {
    System.out.println(declaredField);
}
Field d = personClass.getDeclaredField("d");
d.setAccessible(true);
//暴力反射,忽略访问权限修饰符的安全检查
d.set(p,"李四");
Object o1 = d.get(p);
System.out.println(o1);
//出现异常IllegalAccessException,访问不是public项，需要忽略访问权限修饰符的安全检查

```

2. 获取构造方法
```java
Constructor<?>[] getConstructors()
Constructor<T> getConstructor(类<?>... parameterTypes)
Constructor<?>[] getDeclaredConstructors()


Constructor类的方法：
//用来创建对象
public T newInstance(Object... initargs)
//如果是空参创建对象，可以使用Class的newInstance的方法
```
示例代码
```java
Class personClass = Person.class;
Constructor constructor = personClass.getConstructor(String.class, int.class);
//构造方法也要添加参数的Class
System.out.println(constructor);

Object person = constructor.newInstance("张三", 26);
System.out.println(person);
```

3. 获取成员方法
```java
Method[] getMethods()
Method getMethod(String name,类<?>... parameterTypes)

Method类的方法：
public Object invoke(Object obj,Object... args)
//调用方法


```
示例代码：
```java
Class personClass = Person.class;
Method eat = personClass.getMethod("eat");
//第一个参数是方法名，之后的参数是方法的参数
System.out.println(eat);
Person p = new Person();
eat.invoke(p);
//调用方法

Method[] declaredMethods = personClass.getDeclaredMethods();
for (Method declaredMethod : declaredMethods) {
    System.out.println(declaredMethod);
    //会打印出自己的方法和Object类的方法
    System.out.println(declaredMethod.getName());
    //打印出方法名称
}
```


4. 获取类名
```java
String getName()
```

示例代码

```java
Class personClass = Person.class;
System.out.println(personClass.getName());
```


# 注解

注释：描述程序的文字，给程序员看

注解：1.5版本之后的新特性，说明程序，给计算机看的

作用分类：
1. 编译检查：例如`@Override`
2. 编写文档
    ```java
        /**
        * @author BlackKnight 
        * @version 1.0
        * @since 1.5
        */
    ```
    ```java
         /**
         * 
         * @param a 整数
         * @param b 整数
         * @return 两数的和
         */
        public int add(int a,int b ){
            return a+b;
        }
    ```
    在cmd命令行运行`javadoc 程序名`抽取文件注解
3. 代码分析（使用反射）

## JDK预定义的注解
- @Override

    用来检测被该注解标注的方法是否是继承自父类（接口）的
    ```java
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
            '}';
    }
    ```
- @Deprecated

    将该注解标注的内容标记已过时
    ```java
    @Deprecated
    public void eat(){
        //不建议使用这个方法
        //但是还可以使用
        System.out.println("吃饭");
    }
    ```

- @SuppressWarnings

    压制警告
    ```java
    @SuppressWarnings("all")
    //需要传参，一般传all，压制所有的警告
    ```

## 自定义注解
- 格式：
    ```
    元注解
    public @interface 注解名称{
    }
    ```
    cmd使用`javap 注释类`反编译得到注释的本质

- 本质：

    注解本质上就是一个接口，该接口默认继承Annotation接口
```java
public interface MyAnno extends java.lang.annotation.Annotation {}
```
- 属性：

    接口中的抽象方法

属性的要求：
1. 属性的返回值类型
    - 基本数据类型
    - String
    - 枚举
    - 注解
    - 以上类型的数组
```java
int show1();//属性可以是普通方法
String[] show2();//可以是数组
Enum per();//可以是枚举类型
//可以是另一个注解

//类不可以放入
//void不可以放入
```

2. 定义了属性，在使用时，需要给属性赋值
```java
@MyAnno(show1 = 1)
public class Person {
    //.....
}
```
可以添加默认值，不用赋值
```java
public @interface MyAnno {
    int show1() default 3;
}
```
如果只有一个属性并且属性名称是value，赋值可以直接写值
```java
@MyAnno(1)
public class Person {
    //.....
}
```

## 元注解
用于描述注解的注解
- @Target

    描述注解能够作用的位置
    
    ElemenType取值:
    1. TYPE: 可以作用在类上
    2. METHOD：可以作用于方法上
    3. FIELD：可以作用在成员变量上

- @Retention

    描述注解被保留的一个阶段
    
- @Documented

    描述注解是否被抽取到api文档中

- @Inherited

    描述注解是否被子类继承

```java
@Target({ElementType.TYPE,ElementType.FIELD,ElementType.METHOD})//表示MyAnno注解只能作用在类上
@Retention(RetentionPolicy.RUNTIME)//当前被描述的注解，会保留到class字节码文件中，并被JVM读取到（如果是CLASS不会被JVM读取到）
@Documented
@Inherited
public @interface MyAnno {
}
```

## 程序中使用注解
在程序运行中调用程序的注解 
，用于配置文件

1. 获取注解定义的位置的对象{Class,Method,Field}
2. 获取指定的注解
3. 调用抽象方法中配置的属性值

## 小结
1. 大多数是使用注解而不是自定义注解
2. 注解给谁用？
    1. 编译器
    2. 给解析程序用
3. 注解不是程序的一部分，注解就是一个标签
