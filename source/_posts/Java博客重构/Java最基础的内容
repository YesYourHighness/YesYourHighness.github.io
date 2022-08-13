---
title: Java常量与变量
date: 2019-08-08 21:54:56
tags: 
- Java
categories: 
- Java
---
<center>
引言：认识Java的常量、变量的基本类型、运算符等
</center>

<!--more-->

# 常量

常量类型
1. 字符串常量`'abc','hello'`
2. 整数常量`100,200`
3. 浮点数常量`2.5,3.14`
4. 字符常量:单引号引起来的单个字符`'a','w','2','啊'`可以是汉字，但是不能什么都没有
5. 布尔常量`false,true`
6. 空常量`null`代表没有任何数据(`null`不能用来打印输出)


# 基本数据类型


数据类型|关键字|内存占用 |取值范围
---|---|---|---
字节型|byte|1字节|-128 ~ 127
短整型|short|2字节|-2^15 ~ 2^15-1
整型|int|**4字节**|-2^31 ~ 2^31-1
长整型|long|8字节|-2^63 ~ 2^63-1
单精度浮点型|float|4字节|超大
双精度浮点型|double|8字节|超级大
字符型|char|**2字节**|0~65535(2^16-1)
布尔类型|boolean|**1字节**|true & false

整数型 `byte short int long`

浮点型 `float double`

字符型 `char`

布尔型 `boolean`

注意：
1. 字符串不是基本类型，是引用类型
2. 浮点型可能只是近似一个值，而并非是其精确值
3. 数字范围和字节数不一定相关，例如`float`和`long`
4. 浮点数**默认类型**是`double`，用`float`，**得加F后缀**
5. 整数默认为`int`，用`long`，**要加L后缀**


# 变量

定义变量注意他们的取值范围：
```java
byte x = 10; //正确
byte x = 128; //错误
// byte范围在-128~127之间
```
注意`long`，要加L
```java
long x = 300000000000; // 错误
long x = 300000000000L; //正确
```
注意`float`，要加F
```java
float x = 2.5; //错误
float x = 2.5F; //正确
```
注意事项：
1. 变量名不能重复
2. `float`和`long`要加后缀
3. 未赋值的变量不可以使用
4. 使用`byte`和`short`要注意取值范围
5. 变量不要超出变量的取值范围
6. 一定要先声明后使用，java是从上到下执行的

# 数据类型转换

## 自动类型转换(隐式)
1. 自动完成
2. 数据范围从小到大转换
3. 布尔类型在JAVA中就是布尔，不能运算，不是0,1

## 强制转换
```java
int num = (int)100L; //带上括号就可以实现强制转换
```
注意：
1. 强制类型转换要谨慎使用，易发生精度损失、数据溢出
2. `byte/short/char`都可以数学运算，都会被首先提升为`int`来进行运算
```java
byte x = 60;
byte y = 50;
byte result = x + y;
//这样会报错
//byte + byte -> int + int -> int != byte
```
在运算过程中，他们会变为`int`，可以改成

```java
int result = x + y;
```
或者
```java
byte result = byte(x + y);
```

# 运算符

## 运算符类型
加减乘除 `+-*/`

取模 `%只对于整数有效`

自增自减 `++ --`

赋值运算符 `=`

比较运算符 `> < == <= >=`

逻辑运算符 `&& || !`

三元运算符 `?:`

- 加法多种用法

1. 加法的运算结果是运算范围最大的那一个
2. `char`类型的加法会变为`int`
3. 可以用于字符串相加减

- 逻辑运算符

逻辑运算符`||`和`&&`有短路效果，

如果根据左边已经可以判断得到最终的结果，右侧的代码将不会执行

- 赋值运算符

赋值运算符隐含一个强制的转换类型

# 方法

定义格式
```java
public static void xxx(){
    //方法体  
}
```

调用方法
1. 单独调用 `function()`
2. 打印调用 `System.out.print(function()`
3. 赋值调用 `int x = function`

注意：
`void`的方法只能单独调用，也可以使用`return;`直接返回

同c++，可以根据数据类型的不同和参数的数量不同来实现重载。

唯一的区别--可以通过多类型的**顺序**区别
```java
sum(int a,double b);
sum(double a, int b);
```

# 语句结构

基本内容同c语言

区别：
`switch(x)`其中x只能为
基本数据类型：`byte/short/char/int`
引用数据类型：`String/enum`


# 数组

数组是一种引用数据类型

- 特点：
1. 一个数组中的**数据类型必须一致**
2. 数组的长度在运行期间**长度不可改变**

- 初始化:
1. 动态初始化——指定长度
2. 静态初始化——指定内容，自动推算长度

- 创建：
```java
String[] str;
str = new String[5];//动态
str = new String[]{"a","b","c"};//静态
```
可以化为一步
```java
int[] sum = new int[50];
//动态指定长度创建数组
String[] arr = new String[]{"你好啊","你是谁啊"};
//静态指定数组内容创建数组
```
**当使用动态初始化数组的时候，其中的元素会自动拥有一个默认值**（java的这个特点让我们可以使用数组时不用初始化了）


```
整数-->0
浮点-->0.0
字符-->'\u000'
布尔-->false
引用-->null
```
- 数组长度：直接`.length`即可
```java
System.out.println(arr.length)
```

- 访问：

直接访问数组名，得到该数组对应的**内存地址哈希值**
```
[I@1b6d3586
```
- 传递数组参数
```java
public static void fun(int[] xxx){}
```

- 引用：当一个数组赋值给数组时，他们指向同一片内存
```java
int[] arr = new int[5];
int[] arr2 = arr;
arr[0]=1;
System.out.println(arr[0]);// 1
System.out.println(arr2[0]);// 1
```
- 数组的长度不可变
```java
int[] arr = new int[5];
System.out.println(arr.length);//5
arr = new int[10];
System.out.println(arr.length);//10
//这样看似是更改了数组的长度
//其实是创建了一个新的长度的数组
//更改了原先数组指向的地址
```


-----
# 关于void是不是基本类型

当我们提到java的基本类型时，一般都说是八种。但在Thinking in java 这本java圣经中，将void归为基本类型。

倒也可以说得通：
基本类型和引用类型的区别是：

- **基本类型： 数据在栈中划分内存，值存储在栈上**
- 引用类型： 数据在栈中划分一块内存用来存储其引用（地址），而其真正的数据存放在堆中

而`void`不可以new出来（因为 void 的源码中，将构造函数设置为 private 的，所以外部不能 new 对象），**void也是存储在栈上的**，所以将它归为了基本类型

在Java的Class反射内有一个方法`isPrimitive()` ，可以判断是否是基本类型，当我们输入代码`void.class.isPrimitive()`，它的返回值是true，再一次证明void分为基本类型是有道理的

# static关键字

## 静态成员数据

​		在类中，加了`static`的关键字，意味着该变量不再属于某一个成员，而是属于类本身，该类的所有对象共用一个静态数据

调用：

```java
Person stu1 = new person();
System.out.println(stu1.year);
//不推荐使用对象来调用静态成员数据，这种写法在之后也会被javac翻译为类名称，静态方法名
System.out.println(Person.year);
//推荐这种写法
```

## 静态代码块

```java
static{
    System.out.println("你好啊");
}
```

特点：

1. 静态代码块**只会在类加载时执行唯一的一次**
2. 是用来对属性赋值的
3. 静态内容先于非静态

注意：

1. 静态成员函数只能访问静态成员变量，非静态函数可以访问所有的内容
2. **没有this指针**


# 覆盖重写

方法的覆盖重写(override):

**方法的名称一样，参数列表也一样**此时会发生方法的覆盖重写

特点：因为创建时创建的对象是子类的，所以在调用方法时，一定会先调用子类中覆盖重写的函数

注意：

1. 必须保证名称相同，参数列表相同
2. `@override`写在覆盖重写的函数前，用来检测是否成功的覆盖重写
3. 子类方法的返回值必须**小于等于**父类的返回值范围
4. 子类方法的权限必须大于等于父类方法的权限修饰符（`public`> `protected `> (`default`)(即什么都不写) > `private`）

```java
//假设此类已经投入属于，我们要添加功能，不要对此类进行更改
public class phone {
   public void call(){
       System.out.println("打电话");
   }
   public void send(){
       System.out.println("发送短信");
   }
   public void show(){
       System.out.println("显示号码");
   }
}
```

```java
//新建一个新的类，重写要增加功能的方法
public class newPhone extends phone {
    @Override
    public void send(){
        System.out.println("发送短信");
        System.out.println("发送微信");
    }
}
```

# super

用法：

1. 在子类的成员方法中，**访问父类的成员变量**
2. 子类中访问父类的成员方法
3. 子类的构造方法中，访问父类的构造方法`super();`

# 转型

​		无论是上转还是下转都是为了让类的使用范围和适用范围发生变化，以便操作不同范围的变量或者方法。

## 向上转型

```java
father up = new son();//向上转型
up.cry();//不能使用子类新增的方法
up.teach();
// 向上转型无法使用子类新增的方法，
// 但是可以使用覆盖重写父类的方法
```

> 上转型是指**将子类对象使用父类引用进行引用**。

特点：

- 上转型对象可以操作和使用子类继承或者重写的方法
- 上转型对象**不能使用子类的新方法或者变量**

- **向上转型一定是安全的，因为是小范围想大范围的转换**

## 向下转型

> 与向上转型相反，即是把父类对象转为子类对象：作用也与上转相反

向下转型的应用并不多

特点：

- 是向上转型的一个还原过程，强制转换会导致报错

- **向下转型不安全**，因此应保证原本该对象就是子类的一个内容，不能指鹿为马

什么意思呢？

```java
int num = (int) 10.0;//正确，10.0本身其实就是一个整型
int num = (int) 10.5;//错误，10.5本身是一个浮点型
```

示例

```java
public static void main(String[] args) {
    father up = new son();//向上转型
    son down = (son) up;//向下转型
    down.cry();//可以调用子类的新增方法了
}
```

## instanceof的使用

向下转型的过程中我们可能会失败报错，所以一定要先判断

格式： `对象名 instanceof 类名`

```java
public static void main(String[] args) {
    father f1 = new son();//向上转型
    father f2 = new father();
    son son1 = new son();
    son son2 = (son) f1;//向下转型
    System.out.println(f1 instanceof father);   //true
    System.out.println(f1 instanceof son);      //true
    System.out.println(f2 instanceof father);   //true
    System.out.println(f2 instanceof son);      //false
    System.out.println(son1 instanceof father); //true
    System.out.println(son1 instanceof son);    //true
    System.out.println(son2 instanceof father); //true
    System.out.println(son2 instanceof son);    //true
}
```

# final关键字

final关键字，代表不可改变的

用法：
修饰一个类，方法，局部变量，成员变量

## 修饰类

```java
public  final class father 
```

final类不能有任何的子类（太监类），final类中所有的成员方法不可被覆盖重写

## 修饰方法

```
public final void teach(){
    System.out.println("123");
}
```

不管是类还是方法，`final`和`abstract`只能使用一个

## 修饰局部变量

`final`局部变量修饰完成，则其不能再被更改（类似C++的const）

```java
final int x ;
x = 10;
//第一种赋值方法
final int y = 10;
//第二种赋值方法
```

只需要保证有一次赋值即可

注意：

- 对于基本类型，`final`表示变量的数据不可以改变

- 对于引用类型，`final`表示变量的地址值不可以改变(其中的值可以变)

## 修饰成员变量

由于成员变量有默认值，所以用了final之后必须初始化！

1. 直接赋值
2. 用构造函数赋值

```java
final String name;
father(String str){
    this.name = str;  
}
```

# 权限修饰符

java中有四种修饰符

| 权限符       | public | protected | (default) | private |
| ------------ | ------ | --------- | --------- | ------- |
| 同一个类     | Yes    | Yes       | Yes       | Yes     |
| 同一个包     | Yes    | Yes       | Yes       | No      |
| 不同包子类   | Yes    | Yes       | No        | No      |
| 不同包非子类 | Yes    | No        | No        | No      |

# 内部类

一个类的内部包含另一个类

分类：

1. 成员内部类
2. 局部内部类

## 成员内部类

成员内部类：

格式：

```java
修饰符 class 外部类名称{
    修饰符  class  内部类名称{
    }
}
```

```java
public abstract class father {
    public class fatherIn{
        
    }
}
```

**内部类用外部类可以随意使用，但是外部类调用内部类需要建立内部类的对象。**

### 使用成员内部类

1. 间接方法
   （通过外部类的对象，调用外部类的方法，里面间接在再调用内部类的方法）
2. 直接方法
   （格式：外部类名称.内部类名称  对象名 = new 外部类名称().new 内部类）

```java
father.fatherIn innerClass = new father().new fatherIn();
```

### 使用成员内部类的同名变量

```
public class father {
    int num = 10;
    public class fatherIn{
        int num = 20;
        public void whichNum(){
            int num =30;
            System.out.println(num);//30
            System.out.println(this.num);//20
            System.out.println(father.this.num);//10
        }
    }
}
```

格式：`外部类名.this.变量名`

## 局部内部类

定义在**方法内部**的类

**只能调用方法来使用，离开方法将不能使用它。**


```java
    public static void main(String[] args) {
        father f1 = new father();
        f1.innerFun();
    }
```

主函数

```java
public class father {
    public void innerFun(){
    class innerClass{
        innerClass(){
            System.out.println("局部内部类");
        }
    }
    innerClass inner = new innerClass();
    }
}
```

局部内部类

### 局部内部类的final问题

java8+开始，只要局部变量内部没有发生实质性的变化，那么final是可以省略的

```java
class innerClass{
        final int num = 20;
//            int num = 20;同上
        innerClass(){
            System.out.println("局部内部类");
        }
    }
        innerClass inner = new innerClass();
}
```

原因：`new`创建的对象是在堆内存当中的，局部变量是在栈内存当中的

所以在**方法内的局部内部类，在调用后，方法会入栈出栈，但是对象仍然会存在，所以这个变量的值是不可以改变的**

该对象调用此变量的时候会复制一份给自己使用，但是如果变量发生变化，就会报错

## 匿名内部类

也是一种局部内部类

如果接口的实现类只需要使用唯一的一次，那么这种情况下就可以省略掉该类的定义，改为使用匿名内部类。

接口：

```java
接口名称 对象名 = new 接口名称(){
    //覆盖重写所有抽象方法
}
```

```java
public static void main(String[] args) {
        api obj = new api() {
            @Override
            public void out() {
                System.out.println("匿名内部类，只使用一次");
            }
        };
    }
```

注意：

1. 匿名内部类在创建对象的时候，只能使用唯一的一次
2. 如果想多次使用，请使用实现类
3. 匿名内部类省略了实现类/子类名称，匿名内部类是有名字的，但是匿名对象省略了对象名


# 内部类中权限修饰符的使用

1. 外部类： public/default
2. 成员内部类： public/default/protected/private
3. 局部内部类: default

**default是不需要写的意思**

# 包装类

日常使用的基本数据类型，使用起来方便，但是没有对应的方法来操作这些基本类型的数据。

可以使用一个类，把基本类型的数据装起来，在类中定义一些方法，这个类就叫包装类。

**包装类可以把基本类型变为对象类型**，**方便使用`ArrayList`**，因为`ArrayList`是无法填入基本类型的

| 基本类型 | 对应的包装类 |
| -------- | ------------ |
| byte     | Byte         |
| short    | Short        |
| int      | Integer      |
| long     | Long         |
| float    | Float        |
| double   | Double       |
| char     | Character    |
| boolean  | Boolean      |

只有`int`类型和`char`类型的包装类名字是`Integer`和`Character`这两个，其他全是开头字母大写


## 装箱和拆箱

### 装箱

把基本类型的数据包装到包装类中(基本类型的数据->包装类)

构造方法：(整型举例)

- `Integer(int value)`构造一个新分配的`Integer`对象，他表示指定的`int`值
- `Integer(String s)`构造一个新分配的`Integer`对象，他表示`String`参数所指示的`int`值。传递的字符串必须是基本类型的字符串，否则会抛出异常，eg:“100”正确   "a"错误

静态方法:

- `static Integer valueOf(int i)`返回一个表示指定`int`值的`Integer`实例对象
- `static Integer valueOf(String s)`返回保存指定的`String`的值的`Integer`对象

```java
    public static void main(String[] args) {
        //构造方法
        Integer in1 = new Integer(1);
        System.out.println(in1);// 1
        Integer in2 = new Integer('1');
        System.out.println(in2);// 49
        Integer in3 = new Integer("1");
        System.out.println(in3);// 1

        //静态方法
        Integer in4 = Integer.valueOf(1);
        System.out.println(in4);// 1
        Integer in5 = Integer.valueOf('1');
        System.out.println(in5);// 49
        Integer in6 = Integer.valueOf("1");
        System.out.println(in6);// 1
    }
    // java中注意单引号与双引号
```

### 拆箱

在包装类中去除基本类型的数据（包装类->基本数据类型）

成员方法:

```java
int intValue()
//以int类型返回该Integer的值
```

```
    // 构造方法
    Integer in1 = new Integer(1);
    int i = in1.intValue();
    System.out.println(i);
```

### 自动装箱和拆箱

```java
    // 自动装箱
    Integer in = 1;
    //相当于  Integer in = new Integer(1);手动装箱
    int i = in + 0;
    //相当于in.intValue()
```

自动装箱和拆箱直接会在ArrayList中使用

```java
    ArrayList<Integer> list = new ArrayList<>();
    // ArrayList集合无法直接存储整数，可以存储Integer包装类
    list.add(1);//自动装箱:list.add(new Integer(1))
    int a = list.get(0);//自动拆箱: list.get(0).intValue();
```

## 基本类型与字符串类型之间的相互转换

- 基本类型 -> 字符串
  1. 基本类型 + "" (空字符串)
  2. 使用包装类的静态方法`toString(参数)`,不是Object的toString方法
  3. String类的静态方法`valueOf(参数)`
- 字符串 -> 基本类型
  1. 使用包装类的静态方法`parseXXX("数值类型的字符串")`

```java
// 基本类型 -> 字符串
int i1 = 1;
String s1 = i1 + "";
System.out.println(s1 + 2);//12
String s2 = Integer.toString(1);
System.out.println(s2 + 3);//13
String s3 = String.valueOf(1);
System.out.println(s3 + 4);//14
// 字符串 -> 基本类型
int i2 = Integer.parseInt("-100");
System.out.println(i2 + 5);//-95
int i3 = Integer.parseUnsignedInt("1");
System.out.println(i3 + 6);//7
```

