---
title: Java8新特性1
date: 2020-08-23 17:03:05
tags:
- Java8
- Stream
categories:
- Stream
---

<center>
引言：

Java8新特性
</center>
<!-- more -->


# Java8新特性1



## 新特性概览

1. **`Lambda`表达式**
2. 函数式接口
3. 方法引用与构造器的引用与数组引用
4. **Stream API**



## Lambda表达式



### 基础语法

新的操作符：`->`

​	左边：表达式的参数列表

​	右边：表达式所需要实现的功能，称为Lambda体

```java
/**
 * Lambda表达式基础语法：
 *
 * 新的操作符：->
 * 左边：表达式的参数列表
 * 右边：表达式所需要实现的功能，称为Lambda体
 *
 * 语法格式一：无参，无返回值
 * () -> System.out.println("Hello");
 *
 */
@Test
public void grammerBase1(){
    //原来
    Runnable run1 = new Runnable() {
        @Override
        public void run() {
            System.out.println("hello");
        }
    };

    //使用Lambda表达式
    Runnable r2 = () -> System.out.println("hello");
}

/**
 * 语法格式二：一个输入，无返回值
 * 一个输入可以省略小括号
 */
@Test
public void grammerBase2(){
    Consumer<String> con = x -> System.out.println(x);
    // 一个参数 -> 左侧的小括号可以省略
    // 等于： (x) -> System.out.println(x)
    con.accept("哈哈哈");
}

/**
 * 语法格式三：两个以上的参数，有返回值，有多条语句
 * 参数逗号隔开
 * 返回值使用return返回
 * 多条语句使用大括号包裹
 */
@Test
public void grammerBase3(){
    Comparator<Integer> com = (x,y) -> {
        System.out.println("函数式接口");
        return Integer.compare(x,y);
    };
}
/**
 * 语法格式四：lambda体中只有一条语句
 *
 * return和大括号都可以省略不写
 */
@Test
public void grammerBase4(){
    // 使用grammerBase3的例子
    Comparator<Integer> com = (x,y) -> Integer.compare(x,y);
}

/**
 * 语法格式五：参数的参数类型是否可写
 * Lambda表达式的参数类型可以不写，JVM可以通过上下文推断出数据类型，称为类型推断
 * 类型推断例如：
 *      String strs[] = {"123"}; 这样可以
 *
 *      String strs[];
 *      strs = {"123"};          这样就不行了
 *
 * 这就是因为JVM帮我们做了类型推断，自动推断{}内都是string类型，但是分开写，没有了上下文环境，就推断不出来了
 * 通用的例子还有：
 *      ArrayList<Integer> list = new ArrayList<>();
 *      new 后面的尖括号内我们可以不写泛型，也是因为有着类型推断
 */
@Test
public void grammerBase5(){
    // 使用grammerBase4的例子
    // 如果要写参数类型，那么两个都要写
    Comparator<Integer> com = (Integer x,Integer y) -> Integer.compare(x,y);
}
```

### 注意

**需要函数式接口的支持！！**

所谓函数式接口，就是只含有一个方法的接口，大于一个方法就不能使用Lambda表达式了

可以使用注解`@FunctionalInterface`来检测一个接口是不是函数式接口

例如：

```java
@FunctionalInterface
public interface MyInterface {
    public boolean test(Integer e);
}
```



### 使用

函数式接口

```java
@FunctionalInterface
public interface MyOpration {
    Integer it(Integer num);
}
```



```java
public Integer calculate(Integer num , MyOpration myOpration){
    return myOpration.it(num);
}
```

使用

```java
@Test
public void useLambda(){
    // 实现 平方
    Integer calculate = calculate(5, x -> x * x);
    System.out.println(calculate);
}
```



## 函数式接口

函数式接口不需要我们来实现（除非及其特殊的情况），共有四个内置的核心函数式接口

1. `Consumer<T>`：消费型接口

   - 返回值：`void accept(T e)`
   - 没有返回值，所以叫做消费型接口

2. `Supplier<T>`：供给型接口

   - 返回值：`T get()`
   - 不需要输入，会返回一个对象，所以叫供给型接口

3. `Function<T,R>`：函数型接口

   - 返回值：`R apply(T t)`，R是返回值类型
   - 一个输入，一个输出

4. `Predicate<T>`：断言型接口

   - 返回值：`Boolean test(T)`
   - 返回值是一个布尔类型

   

其实相关的函数式接口还有很多，例如两个参数输入，一个参数输出的类型等等

 

```java
/**
 * 消费型接口：
 * 一个输入，没有输出
 */
@Test
public void consumerTest(){
    lunch(100.0, x-> System.out.println("今天午饭吃了"+x+"元饭"));
}
public void lunch(double money , Consumer<Double> con){
    con.accept(money);
}

/**
 * 供给型接口：
 * 产生指定个数的随机整数，放入集合当中
 */
@Test
public void supplierTest(){
    List<Integer> numList = getNumList(5, () -> (int)(Math.random() * 100));
    numList.forEach(System.out::println);
}
public List<Integer> getNumList(int num, Supplier<Integer> supplier){
    List<Integer> list = new ArrayList<>();
    for(int i =0;i<num;i++){
        list.add(supplier.get());
    }
    return list;
}

/**
 * 函数型接口：
 * 处理字符串
 */
@Test
public void functionTest(){
    String s = strHandler("你好小朋友，\t\t\t", str -> str.trim());
    System.out.println(s);
}
public String strHandler(String str, Function<String,String>fun){
    return fun.apply(str);
}

/**
 * 断言型接口：
 * 处理字符串，将满足条件的字符串放入集合中去
 */
@Test
public void predicateTest(){
    List<String> strings = Arrays.asList("你好", "小朋友", "1", "85", "学习学习学习");
    List<String> res = filterList(strings, s -> s.length() >= 3);
    res.forEach(System.out::println);
}
public List<String> filterList(List<String> list , Predicate<String> pre){
    List<String> strList = new ArrayList<>();
    for (String s : list) {
        if(pre.test(s)){
            strList.add(s);
        }
    }
    return strList;
}
```



## 方法引用与构造器的引用与数组引用



### 方法引用

> 方法引用：
>
> 如果Lambda表达式中的内容已经实现了，我们可以使用方法引用（可以理解为方法引用是Lambda表达式的另外一种表现形式）

语法格式：

1. `对象 :: 实例方法名`
2. `类 :: 静态方法名`
3. `类 :: 实例方法名`



```java
Student libai = new Student("李白", 345);
// 普通的使用方法
Consumer<String> con1 = x -> System.out.println(x);
con1.accept(libai.getName());

// 实例::方法名
Supplier<Integer> sup = libai::getAge;
System.out.println(sup.get());

// 类::静态方法名
//原来： Comparator<Integer> com = (x,y)->Integer.compare(x,y);
Comparator<Integer> com = Integer::compareTo;

// 类::实例方法名
// 使用注意：第一个参数是实例方法的调用者，第二个参数是实例方法的参数时 才 可以使用类::实例方法名
BiPredicate<String,String> bp1 = (x,y) -> x.equals(y);
BiPredicate<String,String> bp2 = String::equals;
```



### 构造器引用

> 构造器引用：
> 使用类的构造器 ClassName::new

```java
@Test
public void constructCiteTest(){
    // 原来
    Supplier<Student> sup1 = ()-> new Student();
    // 构造器
    Supplier<Student> sup2 = Student::new;
}
```

如果构造器有很多，那么调用的构造器会是前面函数的参数的个数

```java
// 调用 Student(String name, Integer age) 方法
BiFunction<String,Integer,Student> fun = Student::new;
```



### 数组引用

> 数组引用：
>
> Type::new



```java
// 正常的lambda
Function<Integer,String[]> fun1 = x -> new String[x];
String[] apply1 = fun1.apply(10);

// 数组引用
Function<Integer,String[]> fun2 = String[]::new;
String[] apply2 = fun2.apply(10);
```



