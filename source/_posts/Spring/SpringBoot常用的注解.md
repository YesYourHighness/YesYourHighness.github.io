---
title: 注解
date: 2021-11-19 18:03:22
tags:
- Spring
- Java
- 注解
categories:
- Spring
- SpringBoot
---

<center>
引言：什么是注解？注解的本质是什么？自己实现一个注解吧！顺便介绍一下项目内Spring的常用的注解
</center>


<!--more-->

# 注解

## Java注解

### 注解的基本概念

> 什么是注解？

几种理解注解的方式：

- **注解是一种标签**：插入到源码中，可以与其他工具配合使用来完成一些工作，它们可以提供用来完整地描述程序所需的信息，而这些信息是无法用Java来表达的
- **注解是一种接口**：注解的本质是一个接口

> 为什么引入了注解？

1. 注解在一定程度上是在把**元数据**与**源代码**文件**结合在一起**，而不是保存在外部文档中这一大的趋势之下所催生的

2. 受到`C#`语言的启发

   （其实就是迫于`C#`语言带来的压力：例如检测是否重写`override`这个功能`C#`有一个关键字，而在`Java`中，`@Override`是可选择的）

### 创建注解

编写注解的语法如下：

```java
// 注解需要使用 @interface 来标记这是一个注解
public @interface MyAnnotation {
    // 在注解内，这些值称为“元素”
    int value() default 0;
    String description() default "";
}
```

可以看到，使用了`@interface`这个标记，并且注解**元素**的定义与定义接口方法类似

创建一个注解：

- 使用`@interface`标记这是一个注解
- 添加元素的语法如同添加一个接口方法（如果一个元素都不给，这种注解称为**标记注解**）
- 使用`default`可以给这个元素加一个默认值

### 注解的本质

我们反编译一下上面的那个注解，`javap -c MyAnnotation.class`，就能看到注解的真实面目：

```java
// 所有的注解默认继承了Annotation
public interface test.annotation.MyAnnotation 
    extends java.lang.annotation.Annotation {
    // 所谓的元素就是一个 public abstract修饰的方法
    public abstract int value();

    public abstract java.lang.String description();
}
```

重点：

- 所有注解默认继承自`java.lang.annotation.Annotation`
- 注解的**元素其实就是一个接口方法**
- **默认值是动态计算而来，没有与注解存储在一起**！！（这也是反编译后我们找不到有关默认值设置的原因）

### JDK的注解

JDK给我们提供了一些注解

#### 父接口Annotation

所有接口都默认继承了这个接口！

这个类`java.lang.annotation.Annotation`有四个方法：

```java
public interface Annotation {
    boolean equals(Object obj);

    int hashCode();

    String toString();
    
    // 返回Class对象
    Class<? extends Annotation> annotationType();
}
```

#### 元注解

> 元注解是JDK提供的，专门用来注解其他注解的注解（绕口令）

有四种元注解：

- `@Target`：标记一个注解可以标记在什么地方
- `@Retention`：标记一个注解的级别
- `@Documented`：将此注解包含在JavaDoc内
- `@Inherited`：允许子类继承父类中的注解



**1、`@Target`可以标记的类型**：

由一个枚举类`ElementType`给出

```java
public enum ElementType {
    TYPE, // 类（包括枚举）、接口（包括注解）
    FIELD,// 域（或者叫属性）
    METHOD,// 方法
    PARAMETER,// 参数
    CONSTRUCTOR,// 构造方法
    LOCAL_VARIABLE,// 局部变量
    ANNOTATION_TYPE,// 注解类型
    PACKAGE,// 包（没错，可以标记包，但是要在特定的一个文件内）
    TYPE_PARAMETER,//JDK1.8新增
    TYPE_USE// JDK1.8新增
}
```

例如这样使用：

```java
@Target({ElementType.METHOD, ElementType.FIELD})
// 可以使用逗号隔开，这样表示我这个注解只能标记方法和属性
// 如果只有一个，可以去掉{}，如下
// @Target(ElementType.TYPE)
public @interface MyAnnotation {
    ...
}
```

如果一个注解没有被`@Target`标记，表示这个注解可以标记在任何地方！

**2、注解的级别**：

同样由一个枚举类`RetentionPolicy`给出：

- `SOURCE`：注解将被编译器丢弃
- `CLASS`：注解在class文件可用，但是会被VM丢弃
- `RUNTIME`：VM会保留这个注解（**标记为RUNTIME才可以用反射获取**）

```java
public enum RetentionPolicy {
    SOURCE,
    CLASS,
    RUNTIME
}
```

#### 其他常见的JDK注解

比如：

1. `@Override`：标记方法使用，可以看到级别为SOURCE，表示这个注解编译时期就会被丢弃

   ```java
   @Target(ElementType.METHOD)
   @Retention(RetentionPolicy.SOURCE)
   public @interface Override {
   }
   ```

2. `@Deprecated`：表示你使用的类或方法等已过期，不推荐使用

   ```java
   @Documented
   @Retention(RetentionPolicy.RUNTIME)
   @Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
   public @interface Deprecated {
   }
   ```

3. `@SuppressWarnings`：关闭不当的编译器警告信息

   ```java
   @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
   @Retention(RetentionPolicy.SOURCE)
   public @interface SuppressWarnings {
       String[] value();
   }
   ```

### 注解小实战

> 要求：实现一个注解，对于标记了这个注解的方法在运行时会打印Hello 加对应的参数！

注解：

```java
@Target(ElementType.METHOD)// 标记方法
@Retention(RetentionPolicy.RUNTIME)// RUNTIME才可以被反射获取
public @interface Hello {
    String value() default "";
}
```

给方法加注解：

```java
public class User {
    @Hello("小明")
    public void xiaoMing(){}
    @Hello("小军")
    public void xiaoJun(){}
}
```

处理器：

```java
public static void main(String[] args) {
    User user = new User();
    Method[] methods = user.getClass().getDeclaredMethods();
    for (Method method : methods) {
        Hello ann = method.getAnnotation(Hello.class);
        if (ann != null) {
            // 非空说明此方法有注解
            System.out.println("Hello" + ann.value());
        }
    }
}
/* 输出：
    Hello小明
    Hello小军
*/
```

## Spring注解

### Bean相关注解

> 交给Spring管理

- `@Component`：下面三个其实与上面的`@Component`一样，只是分工明确

- `@Service`：标记业务层，业务层经常需要注入Dao层的Bean
- `@Repository`：标记Dao层，表明这个类可以实现CRUD的功能
- `@Controller`：标记Controller，需要注入Service的Bean
- `@RestController`：此注解相当于`@ResponseBody`与`@Controller`
- `@Configuration`：配置类

> 依赖注入

- `@Autowired`：默认按照类型**ByType**匹配
  - 与`Qualifier("name")`配合使用，可以实现与`@Resource`一样的按名称检索的功能
  - 加上`required=false`，表示找不到Bean也不会报错
- `@Resource`：默认按照名称**ByName**装配（此注解并不属于Spring，而是JDK自带的注解）

> Bean的作用域

`@Scope`：标记Bean的作用范围

- `singleton`单例模式：全局有且仅有一个实例（**默认就是单例模式**）
- `prototype`原型模式：每次获取Bean的时候会有一个新的实例
- `request `：表示该针对每一次HTTP请求都会产生一个新的`bean`，**仅在请求中有效**
- `session` ：表示该针对每一次HTTP请求都会产生一个新的`bean`，**在整个会话内有效**
- `globalsession` ：类似于标准的HTTP Session作用域，整个`web`应用有效

### HTTP请求

SpringMVC的注解：

- `@RequestMapping` 注解，有三个元素：
  - `name`：为映射地址指定别名
  - `value`：**默认属性**，映射请求到一个方法或类上
  - `method`：可以选择 `POST`、`GET`等请求方式
- `@GetMapping`：相当于`@RequestMapping(method=RequestMethod.GET)`，下面三个同理
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`

### 配置启动

- `@ComponentScan`

  配置扫描包的路径，相当于你之前在 `xml`中配置 `bean`

- `@Configuration`

  允许在上下文中注册额外的bean或导入额外的配置类

- `@SpringBootApplication`：SpringBoot的启动类需要添加的注解

  这个注解相当于三个注解：

  - `@EnableAutoConfiguration`：启用自动装配机制（即如果你有）
  - `@ComponentScan`
  - `@SpringBootConfiguration`：其实就是`@Configuration`注解

- `@EnableAutoConfiguration`：**类级别的注解**，这个注解告诉 `Spring Boot` 根据添加的`jar`依赖猜测你想如何配置`Spring`

  **非侵入式**自动配置：即SpringBoot会自动为你装配，如果你有自己的配置，会优先使用你的配置





## 参考资料

1. 《Java编程思想》一如既往的牛！
2. 《Java核心技术 卷二》讲的略差一些