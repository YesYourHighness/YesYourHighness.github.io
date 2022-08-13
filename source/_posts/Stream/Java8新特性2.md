---
title: Java8新特性2
date: 2020-08-23 17:04:05
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

# Java8新特性2



## 新特性概览

1. **`Lambda`表达式**
2. 函数式接口
3. 方法引用与构造器的引用与数组引用
4. **Stream API**



1、2、3、请看 Java8新特性1




## 强大的StreamAPI

> Stream 流：
>
> 是一个数据渠道，可以对数据源（集合、数组）的数据进行一系列的操作。
>
> （集合讲的是数据，流讲的是计算）

注意：

1. Stream 不会存储元素
2. Stream 不会改变源操作对象
3. Stream 是延迟执行的，等到需要的结果出现后才会执行



步骤：

1. 创建Stream
2. 中间操作
3. 终止操作（中断操作）

![image-20200823091700968](http://img.yesmylord.cn//image-20200823091700968.png)

### 创建Stream流的四种方法

```java
 //1. 可以使用Collection集合提供的Stream()或parallelStream() （一个是串行流、一个是并行流）
ArrayList<String> strings = new ArrayList<>();
Stream<String> stream = strings.stream();

//2. 可以使用Arrays的静态方法stream获取流
Integer[] integers = new Integer[10];
Stream<Integer> stream1 = Arrays.stream(integers);

//3. 通过Stream类中的静态方法of
Stream<Integer> stream2 = Stream.of(integers);

//4. 创建无限流：会一直生成
// 迭代
Stream<Integer> stream3 = Stream.iterate(0, (x) -> x + 2);
// 需要两个参数：T seed 种子，UnaryOperator单元运算，继承了Function，给一个T，返回一个T
// 可以加上中间操作来终止
stream3.limit(10);

// 生成
// 参数是一个Supplier
Stream<Double> generate = Stream.generate(() -> Math.random());
generate.limit(10);
```



### Stream的中间操作

#### 筛选与切片：

1. `fiter`——接收lambda，从流中排除某些元素
2. `limit`——截断流，使其元素不超过给定的数量
3. `skip(n)`——跳过元素，返回一个去除了前n个元素的流。若流元素不足n个，则返回一个空流，**与limit(n)互补**
4. `distinct`——筛选，通过流所生成的元素的hashCode()和equals()**去除重复元素**

```java
// 中间操作不会进行任何的处理，而在终止操作时，一次性全部处理。（称为：惰性求值）
// 中间操作：
Stream<Student> studentStream = students
        .stream()
        .filter(x -> {
            System.out.println("中间操作");
            return x.getAge() > 30;
        });
// 终止操作：一次性执行全部内容
studentStream.forEach(System.out::println);

// limit
Stream<Student> stream1 = students.stream().limit(2);
stream1.forEach(System.out::println);
//skip
Stream<Student> stream2 = students.stream().skip(2);
stream2.forEach(System.out::println);
// distinct去重，需要重写hashCode和equals方法
Stream<Student> stream3 = students.stream().distinct();
stream3.forEach(System.out::println);
```



#### 映射

> `map`——接收一个lambda，将元素转换为其他形式或提取信息。提取一个函数作为参数，该函数会被应用到每一个元素上，并将其映射为一个新的元素。
>
> `flatMap`——接收一个函数作为参数，将流中的每一个值都换成另一个流，然后把所有流连接成一个流

```java
List<String> strings = Arrays.asList("aaa", "bbb", "ccc");
Stream<String> stream1 = strings.stream()
        .map(x -> x.toUpperCase());
// map 是将该 流 添加到流中去
stream1.forEach(System.out::println);

System.out.println("-------flatMap------");
// 返回一个流
Function<String,Stream> fun = (x) -> strings.stream();
// flatMap使每一个流中的元素都添加到新流中
Stream stream2 = strings
        .stream()
        .flatMap(fun::apply);
stream2.forEach(System.out::println);
System.out.println("--------map---------");
// 对比map 会让流这个对象添加到新流中
Stream stream3 = strings
        .stream()
        .map(fun::apply);
stream3.forEach(
System.out::println);
```

运行结果如下：

<img src="http://img.yesmylord.cn//image-20200823112833993.png" alt="image-20200823112833993" style="width: 50%;" />



#### 排序

>排序：
>
>`sorted()`——自然排序（Comparable），按照字典序
>
>`sorted(Comparator com)`——定制排序（Comparator）

排序操作：

```java
List<String> strings = Arrays.asList("ccc", "bbb", "aaa");
strings.stream().sorted().forEach(System.out::println);
// 按字典序排列

// 自定义排序：先按年龄排序，再按姓名排序
students.stream()
        .sorted((e1,e2)->{
            if(e1.getAge().equals(e2)){
                return e1.getName().compareTo(e2.getName());
            }else {
                return e1.getAge().compareTo(e2.getAge());
            }
        })
        .distinct()
        .forEach(System.out::println);
```



### 终止操作

#### 查找与匹配：

> 查找与匹配操作：
>
> `allMatch` ——是否匹配所有元素
>
> `anyMatch`——是否至少匹配一个元素
>
> `noneMatch`—— 是否没有匹配所有元素
>
> `findFirst`——返回第一个元素，返回值是一个`Optional`对象，该对象可以防止为null
>
> `findAny`——返回当前流中的任意元素
>
> `count`——返回流中元素的总个数
>
> `max`——返回流中最大值
>
> `min`——返回流中最小值

数据：

```java
List<Student> students =  Arrays.asList(
        new Student("李白",15),
        new Student("杜甫",27),
        new Student("白居易",96),
        new Student("孟浩然",77),
        new Student("孟浩然",77),
        new Student("孟浩然",77)
);
```



```java
// 是否所有人的年龄都是27岁？
boolean b1 = students.stream()
        .allMatch(x -> x.getAge().equals(27));
System.out.println(b1);// false
// 是否有人的年龄是27岁？
boolean b2 = students.stream()
        .anyMatch(x -> x.getAge().equals(27));
System.out.println(b2);// true
// 是否没有人的年龄是27岁？
boolean b3 = students.stream()
        .noneMatch(x -> x.getAge().equals(27));
System.out.println(b3);// false

Optional<Student> first = students.stream().sorted((e1, e2) -> {
    return -e1.getAge().compareTo(e2.getAge());
}).findFirst();
//first.orElse( other ); 这个方法的意思是：如果这个类为空，就可以使用other来替代
System.out.println(first);


Optional<Student> any = students.parallelStream().findAny();
System.out.println(any);

// 返回流中的个数
long count = students.stream().count();

// max和min中的参数也是predict
Optional<Student> max = students.stream().max((e1, e2) -> {
    return e1.getAge().compareTo(e2.getAge());
});
System.out.println(max);
```



#### 归约与收集

> 归约：
>
> `reduce(T identity, BinaryOperator)`
>
> `reduce(BinaryOperator)`
>
> 可以将流中的元素反复结合起来得到一个值。



```java
// 常配合 map 使用，这里将Student中每一个元素的年龄相加起来
Optional<Integer> reduce = students.stream()
        .map(Student::getAge)
        .reduce(Integer::sum);
System.out.println(reduce.get());
```

`map-reduce`组合常用在大数据当中



> 收集：
>
> `collect`——将流转换为其他形式，接收一个`Collector`接口的实现，用于给`Stream`中元素做汇总的方法

```java
// 把所有的名字放到list中
List<String> collect = students
        .stream()
        .map(Student::getName)
        .collect(Collectors.toList());
collect.forEach(System.out::println);

// 把所有的名字放到Hashset中
HashSet<String> collect1 = students.stream()
        .map(Student::getName)
        .collect(Collectors.toCollection(HashSet::new));
collect1.forEach(System.out::println);
```

收集还有很多种方式，可以求最大最小等等，这里不再细说。





### 并行流

> 并行流：
>
> 将一个内容分为多个数据块，使用不同的线程分别处理每个数据块的流，并且含有工作窃取，即一个核对应的任务完成后，会从其他核偷取任务来执行，大大提高了执行效率
>
> 使用`parallel`切换为并行流
>
> 使用`sequential`切换为串行流

在之前有一个`ForkJoin`框架，这个框架就实现了对一个任务的拆分与合并，大大提高了处理速度。

在Java8之后我们可以使用并行流来处理（底层其实还是ForkJoin框架）



```java
Instant start = Instant.now();
LongStream.range(0,100000000000L)
    .parallel()
    .reduce(0,Long::sum);
Instant end = Instant.now();
System.out.println(Duration.between(start,end).toMillis());
```

运算一千亿个数循环相加，用时12s，下图可见我的CPU利用率基本占满

<img src="http://img.yesmylord.cn//image-20200823164628042.png" alt="image-20200823164628042" style="width:50%;" />