---
title: Java反射_检查类结构
date: 2020-03-01 22:52:18
tags:
- 反射
categories:
- Java
- 反射
---

<center>
引言：反射机制
</center>

<!-- more -->

# 使用反射类
我们使用反射类主要就就是为了检查类的结构

一个类主要由三个部分组成
- Field 字段域
- Method 方法
- Constructor 构造器

## Field

主要有四个方法，以Filed为例
```
Field getField(name)

Field getDeclaredField(name)

Field[] getFields()

Field[] getDeclaredFields()
```

`Class`类中的 `getFields`、 `getMethods` 和 `getConstructors`方法将分别返回类提供的 `public`域、 方法和构造器数组， **其中包括超类的公有成员**

`Class` 类的 `getDeclareFields`、 `getDeclareMethods` 和 `getDeclaredConstructors`方法将分别返回类中声明的全部域、 方法和构造器， **其中包括私有和受保护成员，但不包括超类的成员**

区别就是：
1. `getField()`只能返回`public`的字段、包括继承自父类的字段
2. `getDeclareField()`返回全部字段，不会返回继承的字段

例如：
```java
class Student extends Person {
    public int score;
    private int grade;
    protected int edu;
    int id;
}

class Person {
    public String name;
}
```
```java
public static void main(String[] args) throws Exception {
    Class cls = Student.class;
    Field[] fields1 = cls.getFields();
    System.out.println(Arrays.toString(fields1));
    //[public int Student.score, public java.lang.String Person.name]
    Field[] fields2 = cls.getDeclaredFields();
    System.out.println(Arrays.toString(fields2));
    //[public int Student.score, private int Student.grade, protected int Student.edu, int Student.id]

}
```



---


获得到`Field`对象之后，这个字段的任意信息我们都可以查看了
```
public String getName()
//返回字段名称
public Class<?> getType()
//返回字段类型，也是一个Class实例，例如，String.class；
public int getModifiers()
//返回字段的修饰符，它是一个int，不同的bit表示不同的含义，可以使用Modifier类的静态方法来解析
```
我们尝试了解`String`类的`value`字段
```java
public final class String {
    private final byte[] value;
}
```
```java
Field f = String.class.getDeclaredField("value");
f.getName(); // "value"
f.getType(); // class [B 表示byte[]类型
int m = f.getModifiers();
Modifier.isFinal(m); // true
Modifier.isPublic(m); // false
Modifier.isProtected(m); // false
Modifier.isPrivate(m); // true
Modifier.isStatic(m); // false
```
----

上述只是第一步，我们更多时候是想得到实例的字段值

我们要读取的类
```java
class Student extends Person {
    public int score;
    private int grade;

    public void setGrade(int grade) {
        this.grade = grade;
    }

    public int getGrade() {
        return grade;
    }

    public Student(String name) {
        super(name);
    }
}

class Person {
    public String name;

    public Person(String name) {
        this.name = name;
    }
}
```

```java
Student stu1 = new Student("Jack");
Student stu2 = new Student("Rose");
//------------分隔----------
Class cls = Student.class;
Field name = cls.getField("name");
Object o1 = name.get(stu1);
Object o2 = name.get(stu2);
System.out.println(o1);//Jack
System.out.println(o2);//Rose
```
核心是调用Field实例的`get()`方法

如果我们访问一个`private`字段呢？

```java
Student stu1 = new Student("Jack");
Student stu2 = new Student("Rose");
stu1.setGrade(1);
stu2.setGrade(2);
//------------分隔----------
Class cls = Student.class;

Field grade = cls.getDeclaredField("grade");
//获取要用getDeclaredField方法
Object o = grade.get(stu1);
System.out.println(o);   
```
结果报错了
```
Exception in thread "main" java.lang.IllegalAccessException:...
```
这是因为正常情况下，`private`标记的字段是不能被其他类访问的

修复错误我们可以调用`setAccessible`方法，设置后，我们就可以访问任意类型了
```
Field grade = cls.getDeclaredField("grade");
grade.setAccessible(true);//加上这个
Object o = grade.get(stu1);
System.out.println(o);//1
```
> `setAccessible(true)`可能会失败。如果JVM运行期存在`SecurityManager`，那么它会根据规则进行检查，有可能阻止`setAccessible(true)`。例如，某个`SecurityManager`可能不允许对`java`和`javax`开头的`package`的类调用`setAccessible(true)`，这样可以保证JVM核心库的安全。

可以访问了，当然就可以赋值了

```java
Field score = cls.getDeclaredField("score");
score.set(stu1,99);
```
核心是`set(object,object)`方法，第一个参数输入实例、第二个参数输入待修改的值（注意类型要匹配，否则会报错）同样如果要访问私有字段，依然要设置`setAccessible(true)`


## Method

获得Method
```java
Method getMethod(name, Class...)
//获取某个public的Method（包括父类）
Method getDeclaredMethod(name, Class...)
//获取当前类的某个Method（不包括父类）
Method[] getMethods()
//获取所有public的Method（包括父类）
Method[] getDeclaredMethods()
//获取当前类的所有Method（不包括父类）
```
基本和Field相同，但在调用时要注意参数，例如：
```java
class Student{
    private String name;
    private int id;

    public void nothing(){
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }
}
```
```java
public static void main(String[] args) throws Exception {
    Student stu1 = new Student();
    //调用有参方法
    Method method1 = Student.class.getMethod("setName",String.class);
    Method method2 = Student.class.getMethod("setId", int.class);
    //调用无参方法
    Method method3 = Student.class.getMethod("nothing");
}
```
----

使用Method

使用反射实现`subString`功能
```java
String name = "hello world";
Method method = String.class.getMethod("substring", int.class);
//Method method = String.class.getMethod("substring", int.class,int.class);
//改变参数即可选择是哪个重载函数
System.out.println(method.invoke(name, 6));//world
```
核心是`invoke()`方法
```java
public Object invoke(Object obj,
                     Object... args)
              throws IllegalAccessException,
                     IllegalArgumentException,
                     InvocationTargetException
/*第一个参数是这个方法所操作的对象，第二及以后的参数都是原方法的参数
（待补充）
*/
```

访问非public方法，记得设置许可


## Constructor
> 调用`Class.newInstance()`的局限是，它只能调用该类的`public`无参数构造方法。如果构造方法带有参数，或者不是`public`，就无法直接通过`Class.newInstance()`来调用。

获取`Constructor`依然使用`getConstructor`或`getDeclareConstructor()`方法

这种情况我们可以使用反射来解决
```
Constructor<Integer> constructor = Integer.class.getConstructor(int.class);
Integer integer = constructor.newInstance(5);
```
核心在于`newInstance()`方法，参数传递构造方法需要传递的参数

同样访问非public方法，记得设置许可





> 参考资料
[廖雪峰官方网站](https://www.liaoxuefeng.com/wiki/1252599548343744/1255945147512512)
官方API
《Java核心技术卷一》