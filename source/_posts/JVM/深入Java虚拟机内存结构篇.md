---
title: 深入Java虚拟机内存结构篇
date: 2021-07-24 17:20:20
tags:
- JVM
categories:
- JVM
---
<center>
  引言:
  JVM内存结构有关内容，不包括GC部分。

</center>

<!-- more -->

# 深入Java虚拟机内存结构篇

> 前言：Java真正的核心从这里开始

## JVM体系

### JVM特点

> 语言无关性:
>
> Java——跨平台的语言
>
> Java——跨语言的平台

Java虚拟机不关心你是否使用了Java语言编写，她**只关心Class文件**

只要这个class文件符合规范，那么他就可以运行，至于是Java还是Scala还是其他，no care 

所以说，**JVM可以不可以运行其他语言编写的程序呢？**

答案：可以的，只要该语言的编译器生成合乎规范的class文件即可

### Java代码执行流程

基本流程：

1. 编写Java源码：获得`.java`文件

2. Java编译器对 1 生成的文件进行编译

   1. 词法分析
   2. 语法分析
   3. 语法 / 抽象语法树
   4. 语义分析
   5. 注解抽象语法树
   6. 字节码生成器，生成`.class`文件

3. Java虚拟机，读取`.class`文件

   1. 类加载器

   2. 字节码校验器

   3. **两种**执行方式：
- 翻译字节码（解释执行）
      
- JIT编译器（编译执行）

### JVM指令集架构

> 指令集架构

有两种指令集架构：

- **栈式架构**：
  - 设计和实现简单，全部使用零地址指令分配
  - 不需要硬件支持，更好跨平台
  - 指令集小
  - 执行性能没寄存器架构高
- 寄存器架构：类似于X86汇编语言
  - 依赖硬件，不同公司产的CPU可能指令集就不同，例如X86和MIPS，就是两种指令集

所以Java为了实现跨平台，就使用了第一种栈式架构

![栈式架构示例图](http://img.yesmylord.cn//image-20210713174821755.png)

### JVM生命周期

1. JVM的启动

启动：由引导类加载器`BootStrap class loader`创建一个初始类来实现JVM的启动

2. JVM的执行

JVM启动的唯一原因是要执行Java程序，但对于操作系统来说，没有Java程序，我们运行的全部都是JVM进程

3. 虚拟机的退出

有以下几种情况：

- 正常执行结束
- 执行过程中遇到了异常、错误而终止
- 操作系统叫停
- 某线程调用`Runtime`类或`System`类的`exit`方法，或`Runtime`类的`halt`方法，并且Java安全管理器也允许本次的操作
- 除此外，还有JNI(Java Native Interface)规范描述的一些情况

### HotSpot

> HotSpot到底是什么？

1. 一种VM实现方式：

2000年，JDK1.3发布，Java HotSpot virtual machine正式发布

HotSpot VM 是Sun JDK与Open JDK默认的JVM

**采用解释器与即时编译器JIT并存的结构**

目前，HotSpot VM 是广泛的JVM实现，主要学习的也就是这个！

2. 一种技术——**热点代码探测技术**

Java原先是把源代码编译为字节码在虚拟机执行，这样执行速度较慢。

而**HotSpot将常用的部分代码编译为本地（原生，native）代码**，这样显着提高了性能。

## JVM结构

### Jvm简略结构

![JVM总结构略图](http://img.yesmylord.cn//image-20210713130642171.png)

> 基本结构

- Class Loader：类加载器子系统
- Runtime Data Areas：运行时数据区
- Execution Engine：执行引擎

## Class Loader 类加载器子系统

### Class Loader 作用

1. 负责从文件系统或网络中**加载Class文件**
2. 只负责加载，不确保可以运行（Execution Engine决定）
3. 加载的类信息存放在 Method Area 方法区

---

我们编写一个`.java`文件的类，例如`Car`，到在JVM中创建实例，有这几个过程：

1. `.java` >>>>`.class`
2. `.class`>>>> 进入JVM >>>> 生成 **DNA元数据模板**
3. JVM根据**DNA元数据模板**生成多个`Car`对象

Class Loader在这个过程中相当于一个快递员

### 类的加载过程

总共有三大步：

1. 加载 **Loading**
2. 链接 **Linking**
3. 初始化 **Initialization**

#### Loading 加载

1. 通过一个类的**全限定名**获取此类的**二进制字节流**

> PS：类的全限定名，即类似cn.yesmylord.www.Hello 这样的

2. 将这个字节流所代表的 **静态存储结构** 转化为 **方法区**的 **运行时数据结构**
3. 在内存中**生成一个代表这个类的`java.lang.Class`对象**，作为方法区对这个类的各种数据的访问入口

---

对于加载来源的补充， `.class`文件可以来自：

- 本地直接加载
- 网络获取
- zip压缩包直接读取的（jar、war格式的基础）
- 运行时自动生成（动态代理技术）
- 其他文件生成（例如JSP文件）
- 从加密文件中获取（防止反编译）

#### Linking 链接

链接有三个步骤：

1. 验证 verify

目的：为了保证`.class`文件内容符合当前虚拟机的规范

（例如我是HotSpot VM 你不能给我 Taobao Vm的字节码文件，这样我读不懂）

包括四种验证：文件格式验证、元数据验证、字节码验证、符号引用验证

---

2. 准备 Prepare

为**类变量**分配内存，并**设置默认初始值**（此时并不会给真正的值，初始化时才会给真正的值）

注意：

- 对于`final`修饰的类变量，在编译时就分配了内存，在此阶段只是显式初始化
- 不会为**实例变量**初始化，因为实例变量会随对象一起分配到**堆区**中

> PS：
>
> - 类变量：即类的变量，即用static修饰的变量，**类变量的信息会放在方法区中**
> - 实例变量：对象的变量，没有使用static修饰，会存放在Java**堆**中

---

3. 解析 Resolve

目的：将常量池中的**符号引用**转换为**直接引用**

（假如程序中用到`System.out.println`，那么符号引用就会引入`java.lang.System`包，紧接着把符号引用转换为直接引用）

注意：

- 实际上，解析过程通常在**JVM初始化完后才会执行**

> ps：
>
> - 符号引用：一组符号描述所引用的目标
> - 直接引用：指针、相对偏移量或一个间接定位到目标的句柄

#### Initialization 初始化

初始化阶段就是执行**类构造器方法`<clinit>()`**的过程

>  ps:
>
> 此方法**不需要定义**，是**javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来**

只要程序中有**静态代码块**或是对于**普通类变量**（指没有用final修饰），就会出现`<clinit>()`方法

![使用idea的jclasslib插件查看](http://img.yesmylord.cn//image-20210713201052604.png)

注意：

- 类构造器**`<clinit>()`方法**不同于类的构造器（即`<clinit>`与`<init>`不同！！**后者每一个类都有，前者则不一定**）

- 类构造器方法**按顺序执行初始化**

  ```java
  public class Hello {
      static {
          temp = 2;
      }
      static int temp = 1;
      public static void main(String[] args) {
          System.out.println(temp); //输出 1
      }
  }
  ```

  ```java
  public class Hello {
      static int temp = 1;
      static {
          temp = 2;
      }
      public static void main(String[] args) {
          System.out.println(temp); //输出 2
      }
  }
  ```

- `final`修饰的类变量不会在`<clinit>`方法中初始化
- 如果该类有父类，会确保父类的`<clinit>`方法先执行
- 一个类的`<clinit>`方法只会在首次使用这个类的时候运行，只运行一次！
- 虚拟机必须保证，一个类的`<clinit>`方法会在并发下被**同步加锁**（即这个方法只会运行一次）

### 类加载器的分类

JVM规范定义，所有派生自ClassLoader的类加载器都划分为自定义类加载器

所以分类有两类：

- **引导类加载器（BootStrap ClassLoader）**：**没有继承ClassLoader类**
- 自定义加载器（User-Define ClassLoader）

最常见的类加载器只有3个：

1. 引导类加载器（BootStrap ClassLoader）
   - **使用C/C++实现**，**在JVM内部**
   - 用来**加载Java核心库**（`JAVA_HOME/jre/lib/rt.jar`、`resources.jar`、`sun.boot.class.path`路径下的内容，用于提供JVM自身运行所需要的类）（比如String类、Integer类等等核心类库）
   - **并没有继承 ClassLoader**，没有父类加载器
   - 加载另两个加载器，是他们的父类加载器
   - 出于安全考虑，只会加载包名为`java, javax, sun`开头的类
   - 如果一个类使用`name.class.getClassLoader`方法获**取到为`NULL`**说明它是由引导类加载器加载的
   
2. 扩展类加载器（Extension ClassLoader）

   - Java语言编写，是`sun.misc.Launcher$ExtensionClassLoader`的**内部类**
   - 间接继承自ClassLoader
   - 从`java.ext.dirs`系统属性指定的目录中加载类库，或从JDK的安装目录`jre/lib/ext`子目录下加载类库。**如果我们写的JAR也放在这里，也会由扩展类加载器自动加载**。

3. 应用程序类加载器（App ClassLoader，也叫系统类加载器 System ClassLoader）

   - Java语言编写，Launcher的内部类
   - **父类为扩展类加载器**
   - **负责加载环境变量classpath或系统属性`java.class.path`指定路径下的类库**

   - 加载自定义的类，可以使用`ClassLoader.getSystemClassLoader()`获取

   ```java
   ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
   System.out.println(classLoader);
   // sun.misc.Launcher$AppClassLoader@18b4aac2 自定义类使用AppClassLoader加载
   ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
   System.out.println(systemClassLoader);
   // sun.misc.Launcher$AppClassLoader@18b4aac2 与上面的相同！！
   ```

### 三个类加载器之间的关系

![image-20210714185226307](http://img.yesmylord.cn//image-20210714185226307.png)

注意：

1. 不是继承关系！！
2. 可以理解为等级制度！！

真正的关系是这样的：

![ClassLoader继承关系](http://img.yesmylord.cn//image-20210714185343482.png)

> 注意：他们俩都是`sun.misc.Launcher`的**内部类**，`Launcher`本身只是一个启动器

### 自定义类加载器

#### 什么时候我们才会要去自定义类加载器？

- **隔离加载类**（防止用中间件导致命名空间冲突）
- 修改类加载的方式
- 扩展加载源
- 防止源码泄露（可以对指定的字节码文件进行解密）

#### 实现自定义类加载器的步骤

1. 继承`ClassLoader`类
2. 实现`findClass()`方法（JDK1.2前是重写loadClass方法）
3. 如果没有太复杂的需求，可以继承`URLClassLoader`类，避免自己去编写`findClass()`方法及其获取字节流的方式

`ClassLoader`是一个抽象类，除了`BootStrapClassLoader`，其余全部继承了`ClassLoader`类，它的抽象方法如下：

- **`getParent()`**：返回该类加载器的超类加载器
- **`loadClass(String name)`**：**加载**名称为name的类，返回`java.lang.Class`类
- **`findClass(String name)`**：**查找**名称为name的类，返回`java.lang.Class`类
- `defineClass(String name, byte[] b, int off, int len)`：把字节数组 b 中的内容转换为一个Java类，返回结果为`java.lang.Class`类的实例
- `resolveClass(Class<?> c)`：连接指定的一个Java类

### 获取ClassLoader的途径

```java
//【法一】：通过当前类的Class对象来获取
ToGetClassLoader.class.getClassLoader();
//【法二】：获取当前线程上下文的ClassLoader
Thread.currentThread().getContextClassLoader();
//【法三】：ClassLoader获取AppClassLoader
ClassLoader.getSystemClassLoader();
//【法四】：通过获取AppClassLoader，进而获取ExtensionClassLoader
ClassLoader.getSystemClassLoader().getParent();
```

### 双亲委派机制

> 双亲委派机制：
>
> 加载class文件时，把加载请求逐级向上递交，上级加载器不加载此class时，才会交由低级的加载器加载。

如图：

![双亲委派机制](http://img.yesmylord.cn//image-20210714223831305.png)

当我们加载自定义的一个类时，也会走这么一个流程：

会一级一级向上传递，然后引导类加载器和拓展类加载器都说自己不加载，才会轮到系统类加载器加载。

#### 面试常见问题：如何将自己写的`java.lang.String`导入JVM

答：

​		如果是`java.lang`包下的内容，是无法加载的，因为双亲委派机制的存在，`java.lang`包下的内容全部会被引导类加载器加载。

​		即使使用了自定义的类加载器去加载，规避双亲委派机制，但也会被**沙箱安全机制**拦截，报出安全异常。

> 某[博客](https://blog.csdn.net/Doit_kang/article/details/107159232?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-1&spm=1001.2101.3001.4242)看到如下的表格就直接拿来用了

| 包路径不为`java.lang`    | 包路径为`java.lang`                                          |
| ------------------------ | ------------------------------------------------------------ |
| 通过应用类加载器加载成功 | 当从程序内部加载自定义类时，加载失败，默认加载Java中的String；当从外部加载时（即写自定义类加载器），加载失败，Java加载类时存在检测机制，不允许加载任何包路径以`java.`开头的自定义类 |

#### 双亲委派机制的优势

1. **避免类的重复加载**

   一个类只会由一个加载器加载，不会出现多个加载器加载一个类的情况

2. **保护程序安全，防止核心API被随意更改**

### 其他

> JVM中表示 **两个class对象是否为同一个类** 的两个必要条件

- 类的全限定类名必须一致
- 加载这个类的`ClassLoader`必须相同

---

> JVM必须知道一个类是由哪个加载器加载的！
>
> 对于非启动类加载器加载的类，JVM会将类加载器的**引用**做为类型信息的一部分放在**方法区**
>
> JVM解析一个类型到另一个类型的引用时，JVM需要保证这两个类加载器是相同的

---

> 类的主动使用与被动使用（类的主动使用和被动使用**区别**就在于，**有没有类加载过程中的初始化过程**）

主动使用，有七种情况：

- 创建类的实例
- 访问某个类或接口的静态变量，或者对静态变量赋值
- 调用静态方法
- 反射
- 初始化一个类的子类
- JVM启动时被标明为启动类的类
- JDK 7开始提供的动态语言

## Runtime Data Areas 运行时数据区

### Runtime Data Areas 基本结构

有五大部分：

- **方法区 Method Area**（在JDK1.8后 叫**元数据区**）
- **堆 Heap**
- 程序计数器 Program Count Register (**PC**)
- 本地方法栈 Native Method Stack (**NMS**)
- 虚拟机栈 JVM Stack (**VMS**)

其中，加粗的部分为每个**进程**一份（即**整个JVM只有一个方法区和堆区**），其他部分每个**线程**各有一份，共用方法区和堆区

而且JVM中的线程与操作系统的线程是一一映射关系的

### PC

> PC 程序计数寄存器：不同于CPU内的PC，而是一种模仿的抽象，也叫程序钩子

#### 功能

PC寄存器用来存储指向下一条指令的地址，也即将要执行的指令代码。由执行引擎（Execution Engine）读取下一条指令。

如图：

![PC的功能](http://img.yesmylord.cn//image-20210715173709950.png)

#### 特点

1. 小而精：占很小的一块内存，处于运行速度最快的区域
2. 线程私有，PC与线程共存亡
3. 不会发生OOM（Out of Memory）
4. 记录**当前方法**的JVM指令地址（若执行`Native Method`，则PC是 undefined）

> 当前方法：任何时间线程只能有一个方法在执行，这个方法就在当前**虚拟机栈**栈顶

#### 例子

```java
public class PC {
    public static void main(String[] args) {
        // -1 0 1 2 3 4 5 由 iconst_m1 0 1 2 3 4 5 指令执行
        // 其他使用 bipush 值 指令执行
        int a = 10;     // bipush 10
        int b = 3;      // iconst_3
        int k = a - b;  // isub
    }
}
```

反编译后如下：

![程序反编译后](http://img.yesmylord.cn//image-20210715180146397.png)

左边的红色数字，代表指令地址（**偏移地址**），右边代表具体的指令

PC在这个过程中会存放偏移地址，执行引擎会从PC中读取位置，再去执行指令

#### 两个问题

> 1. 使用PC存储字节码指令地址有什么用？或者说为什么使用PC记录当前线程的执行地址？

因为CPU会不停的切换线程，当切换回此线程时，得知道从哪儿继续任务，所以就使用PC这个结构来存储指令地址

然后字节码解释器就通过改变PC值来明确下一条应该执行什么样的字节码指令

> 2. PC寄存器为什么要设置为线程私有？

如果不设置PC为私有，那么就得把PC内的值存到一个地方，这样切换线程时，需要不停的存PC读PC，增大了切换线程的开销。要是给每个线程一个PC，那么切换就消除了这个开销。

### JVM Stack

> 一个硬骨头

#### 功能

保存程序运行期间的局部变量（8种基本数据对象、引用对象的**地址**）、部分结果、参与方法的返回和调用

#### 特点

- 线程私有，与线程共存亡
- 存储的基本单位：**栈帧**（Stack Frame）
- 速度仅次于PC
- 不存在垃圾回收问题，但是存在**Stack Overflow**和**Out Of Memory**异常
- JVM对其有两个操作：入栈、出栈

#### 栈的Stack Overflow 与 OOM

JVM允许栈可以是**固定不变**的，也可以是**动态增长**的：

- 固定不变：线程创建时就指定了具体的大小，如果此线程运行过程中，**超出了最大容量**，那么会报出**Stack Overflow Error异常**
- 动态增长：栈可以自己动态增长，但是在尝试扩展时，**无法申请到足够的内存**，或者在创建线程时，内存达不到申请的要求，就会抛出**OOM异常**

总结就是：超出了栈范围 -> Stack Overflow，无法申请到内存 -> OOM

> 可以使用Java的-Xss参数设置栈内存大小：
>
> java -Xss1m //设置1M的内存；k代表kb ；G代表 Gb

#### 栈帧 Stack Frame

> 栈帧的特点

- 每一个方法对应一个栈帧
- 栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息
- 处于栈顶的栈帧，叫做**当前栈帧**；对应的方法，称为**当前方法**；对应执行这个方法的类，就叫做**当前类**
- **执行引擎**运行的所有字节码指令只对**当前栈帧**进行操作
- 若该方法调用其他方法，则会创建新的栈帧，压入栈中，待其运行完成后，出栈。

注意：

- 不同线程的所含的栈帧不允许相互引用
- 如果方法嵌套使用，内方法的返回值会传回外方法的栈帧
- 方法有两种方式将栈帧弹出：
  - `return`语句
  - `throw`抛出异常

---

> 栈帧的结构

有五个部分：

- **局部变量表** Local Variables （LV）
- **操作数栈** Oprand Stack （或叫表达式栈 OS）
- 动态链接 Dynamic Linking （或叫 指向运行时常量池的方法引用）
- 方法返回地址 Return Address （或叫 方法正常退出或异常退出的定义）
- 一些附加信息

有时也把后三个归为一类，叫**帧数据区**

下面具体来介绍他们：

---

##### 1. 局部变量表

- 是一个**数字数组**（`int byte char short`等均为数字；`bool`转换0表示假，非0表示真；引用类型可以使用地址，也为数字）

- 局部变量表在栈中，栈线程私有，因此不存在数据安全问题

- 局部变量表所需的容量大小在**编译期**就确定下来，保存在方法的`code`属性的`maximum local variables`数据项中，在方法运行期间不会改变。
- 方法嵌套的次数由栈的大小决定。（如果一个方法的参数和局部变量越多，使得局部变量表变大，那么嵌套次数就会变少）
- 局部变量表只在当前方法调用中有效

---

局部变量表还有一个概念就是 **插槽 Slot**，Slot是**局部变量表最基本的存储单元**，他有这样几个特点：

- `byte short char bool`存储前会被转换为`int`
- 32位以内占用一个slot（包括**引用类型** 和 **`returnAddress`**）；64位占用两个slot 如`long double`
- JVM会给每一个槽都分配一个索引，通过这个索引则可以取到值；方法被调用时，它的方法参数和局部变量都会**按照顺序**被复制到局部变量表的一个`slot`上
- 对于**构造方法**和**实例方法**，会自动引入一个**this变量**，放在index为0的插槽（也就是第一个插槽）

举个例子：

```java
public static void main(String[] args) {
    char a = 'c';
    long i = 10000000L;
}
private void test() {
    int b = 10;
    System.out.println(b);
}
```

![slot占用图](http://img.yesmylord.cn//image-20210715224443888.png)

![实例方法this参数](http://img.yesmylord.cn//image-20210715231135515.png)

![构造方法也有this参数](http://img.yesmylord.cn//image-20210715231437173.png)

- **槽也可以重用**，如果过了局部变量的作用域，那么下面的变量会占用此槽

例如：一个实例方法

```java
private void test2() {
    int a = 10;
    int b = 100;
    {
        int c = a+b;
    }
    for (int i = 0 ; i< 10;i++){
        System.out.println(2);
    }
    int d = a + b;
}
```

假如槽不会重用，那么会有几个槽呢？

有6个（this、a、b 、c 、i 、d）

但实际上只有4个：（我怎么数了数是5个？注意看序号，有两个序号为3！）

![slot的重用](http://img.yesmylord.cn//image-20210715233306525.png)

JVM执行这个方法时，执行过程如下：

1. 实例方法`this`来一个槽 (slot: 0)
2. 变量 `a b`各来一个槽 (slot: 1, 2)
3. 有`c`再来一个槽 (slot: 3)
4. 诶，`c`消失了，那就让`i`用`c`的槽吧 (slot: 3)
5. 诶，`i`也无了，变量`d`去用吧 (slot: 3)

---

另外，学习Java基础的时候，我们知道：

**类变量可以不给初值使用，但是局部变量不行**

现在我们知道原因了，因为：

类在加载过程中，有**加载、链接、初始化**三个过程，而第二步链接又有**验证、准备、解析**三个过程，在准备阶段，所有的类变量会被给默认值，到了初始化阶段才会将程序员给变量的值赋值给类变量。

但是对于局部变量来说，一个方法的局部变量表就没这么多过程了，如果没给初始值，系统也不知道这个值是多少，也就没法使用

- 对于GC来说，**局部变量表所直接引用或间接引用的对象，都不会被回收**

所以Java性能调优，局部变量表可以大作文章！

---

##### 2. 操作数栈

基于**数组**实现的栈，也叫**表达式栈**

作用：

- 根据字节码指令，往栈中写入数据和读出数据（即**入栈与出栈**）
- 保存计算的中间过程，作为运算结果的**临时**存储空间
- 操作数栈相当于JVM的**临时工作区**，在方法开始调用时，此栈是空的
- 操作数栈有其**栈深度**，在编译器就已确定，保存在Code属性中，为`max_stack`的值
  - 32bit占一个栈深度，64bit占两个栈深度
- 虽然是基于数组实现的，但不能直接用索引访问，只能进行入栈与出栈操作
- 如果方法**有返回值**，其返回值会被压入**当前栈帧**的**操作数栈**，并更新PC，执行下一条指令
- Java虚拟机的解释引擎是**基于栈的执行引擎**，指的就是操作数栈

**了解**：

一个新技术：**栈顶缓存技术**（ToS Top-of-Stack Cashing）

指：将栈顶的元素全部存储在CPU的寄存器当中

原因：JVM是基于栈式的指令，虽然零地址的使用简单，但是会增大入栈和出栈的次数（即增多了对内存的访问次数），所以提出了此项技术。

---

##### 3. 动态链接（待修改）

在栈帧中的一个**指向运行时常量池**中方法的引用

例如这样一段简单的代码；

```java
int c = 100;
private void test3() {
    int a = 10;
    int b = 0;
    c++;
}
```

先简单的分析一下，局部变量表应该有3个槽：this、a、b，JclassLib如下，重点关注 **7**

```
 0 bipush 10
 2 istore_1
 3 iconst_0
 4 istore_2
 5 aload_0
 6 dup
 7 getfield #2 <Slot.c : I> //这里的#2 就指向了常量区
10 iconst_1
11 iadd
12 putfield #2 <Slot.c : I>
15 return
```

局部的常量区：`#2`指向了常量区，对应又有`#8`和`#37`，然后依次找下去，`#10`中对应的就是这个变量的名字`c`，`#11`代表类型是`I`，也就是`int`

```
Constant pool:
...
#2 = Fieldref           #8.#37         // Slot.c:I
...
#8 = Class              #44            // Slot
...
#10 = Utf8               c
#11 = Utf8               I
...
#37 = NameAndType        #10:#11        // c:I
...
#44 = Utf8               Slot
```



---

##### 4. 方法返回地址

存放调用该方法的PC寄存器的值

一个方法结束，**本质上是当前栈帧出栈的过程**：

- 正常结束：调用者的PC的值作为返回地址
- 出现异常退出：**返回地址**要通过**异常表**来确定，栈帧中一般不会保存这部分信息

两种退出方式的区别就在于，异常退出的方法不会给上层调用者返回任何的信息

---

##### 5. 一些附加信息

为了给程序调试提供支持的信息

#### 关于JVM栈的几个问题

> 1. 举例栈溢出的情况

答：Stack Overflow栈溢出，栈中存放栈帧，每一个栈帧代表一个方法，日常编程中，递归调用方法时，当栈帧累计增加起来，就会导致栈的大小不足，导致栈溢出

> 2. -Xss调整栈大小，就能不出现Stack Overflow吗？

答：当然不能，无论调多大的栈内存，都有可能用完。不过栈越大，能跑的方法也就越多，有时候调整栈变大，会解决Stack Overflow的问题。

> 3. 垃圾回收是否涉及到JVM栈

答：垃圾回收不涉及VMS，只有方法区和堆才设计GC操作。

> 4. 方法中定义的局部变量是否线程安全

答：线程安全：

- 只有一个线程可以操作此数据，必是线程安全的；
- 若有多个线程可以操作此数据，那这个数据就是共享数据，若没有进行同步，则存在安全问题；

`StringBuilder`是一个线程不安全的类（`StringBuffer`是线程安全的）

```java
// 线程安全
private void a() {
    StringBuilder sb = new StringBuilder();
    sb.append(1);
    sb.append("2");
}
// 线程不安全
private void b(StringBuilder sb) {
    sb.append(1);
    sb.append("2");
}
// 线程不安全
private StringBuilder a() {
    StringBuilder sb = new StringBuilder();
    sb.append(1);
    sb.append("2");
    return sb;
}
```

`a`方法中是线程安全的，因为`a`中的`StringBuilder`始终都是在VMS内的，每个线程的VMS是独占的，所以也就**不存在**线程安全问题；

`b`方法中线程不安全，因为`b`中的`StringBuilder`是一个引用，这个对象存在的位置在堆中，堆不是线程独占的，所以有可能多个线程争抢，就**存在**线程安全问题；

`c`方法线程不安全，因为它将`StringBuilder`返回出去（逃逸），没有完完全全在VMS内产生，又在VMS内消亡，所以会有线程安全问题

总结：方法内什么样的局部变量不会出现线程安全问题呢？**就是既在方法内产生，又在方法内消亡的局部变量。**



### Native Method Stack

`VMS`用来管理Java方法调用，`NMS`来管理本地方法调用

作用：

- 登记方法中使用到的本地方法

注意，当调用本地方法时，就不再受虚拟机控制了！

---

### Java Heap

> 另一块硬骨头

#### 特点

- **进程共有**，一个JVM只有一个堆内存
- JVM**最大**的一块内存空间
- 内存**大小可以调节** ，**物理上不必连续，逻辑上连续**（使用参数`-Xms10m -Xmx20m`设置堆最小10m，最大20m）
- 线程可以在此划分**私有的缓冲区**（Thread Local Allocation Buffer ，**TLAB**）
- 方法结束后，堆中的对象不会被马上移除，只有GC时才会移除

#### 堆内存结构

> JDK 1.7 堆空间

逻辑上堆分为：年轻代、老年代、**永久代**

- 年轻代：又可以分为两部分
  - Eden 伊甸园区
  - Survivor区
    - `Survivor 0`区
    - `Survivor 1`区
- 老年代
- 永久代：**不属于堆空间**的一部分，只是逻辑上分到了这一部分

> JDK 1.8 堆空间

逻辑上堆分为：年轻代、老年代、**元空间**

年轻代与老年代没有变化

- 元空间：物理上在**直接内存**内，不在堆中

#### 堆内存的设置

堆内存在JVM建立时就确定了，可以通过参数来设置**堆空间（新生代+老年代）**大小：

```
-Xms 表示堆区的起始内存 等价于-XX:InitialHeapSize
-Xmx 表示堆区的最大内存 等价于-XX:MaxHeapSize
```

如果堆区内存超过设置的最大内存，就会出现OOM错误

通常设置两个值为**相同的值**，是为了GC清理完堆区后，不需要重新分隔计算堆区的大小，从而提高性能

默认的初始化值，按电脑的内存不同而不同，大致关系如下：

- `起始内存的值 =  电脑内存大小 / 64`
- `最大内存的值 = 电脑内存大小 / 4`

查看自己JVM堆内存的Demo

```java
public class HeapMem {
    public static void main(String[] args) {
        // 查看堆空间大小
        long initialMemory = Runtime.getRuntime().totalMemory();
        long maxMemory = Runtime.getRuntime().maxMemory();
        System.out.println("-Xms: "+ initialMemory / 1024 / 1024 + "M");
        System.out.println("-XmX: "+maxMemory / 1024 / 1024 + "M");
        System.out.println("系统内存大小（用-Xms来计算）" + initialMemory * 64.0 / 1024 + "G");
        System.out.println("系统内存大小（用-Xmx来计算）" + maxMemory * 4.0 / 1024 + "G");
    }
}
```

运行结果：（我的电脑是 16 GB）

```
-Xms: 245M
-XmX: 3614M
系统内存大小（用-Xms来计算）1.6089088E7G
系统内存大小（用-Xmx来计算）1.4804992E7G
```

注意这里得到的值，**并不是实际的堆空间大小**，它只包括**老年代与部分新生代（伊甸园区 + 任一个Survivor区）**，因此会小一些

可以证实一下：

给这个demo睡一会儿

```java
public class HeapMem {
    public static void main(String[] args) {
        ...
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

设置参数启动

```
-Xms600m
-Xmx600m
```

打开`cmd`

```java
输入jps
```

运行结果如下：

```
-Xms: 575M
-XmX: 575M
系统内存大小（用-Xms来计算）3.76832E7G
系统内存大小（用-Xmx来计算）2355200.0G
```

打开cmd，输入如下内容，红色圈住为新生代、蓝色为老年代，其中后缀为U的为已使用，后缀为C表示总量

![jstat查看内存](http://img.yesmylord.cn//image-20210718200145808.png)

我们加一下`( S0C + S1C + EC + OC )/ 1024 = 600M`

`SOC / 1024 = 25M` ，输出结果少了25M，验证了猜想

（也可以直接用**`-XX:+PrintGCDetails`参数**来打印内容）

#### 新生代与老年代

- 默认比例：新生代 : 老年代  = 1 : 2

可以通过参数进行设置 `-XX:NewRatio=n`其中n表示一个数字，假如为5，那么新生代与老年代比例就为1:5（开发中不会修改这个参数）

新生代的大小可以用参数`-Xmn`显示指定，而且优先度大于上面的选项

- 新生代中，Eden与另外两个Survivor区的比例是`8:1:1`

这个数值也可以调整：`-XX:SurvivorRatio=8`

但去验证一下，会发现其实并不是完全的`8:1:1`，因为默认开启**自适应**，JVM会自动进行调整（但就算显示关闭自适应，也不会是`8:1:1`，只有显示声明参数设`-XX:SurvivorRatio=8`，才会是`8:1:1`）

- **几乎所有**的Java对象都在Eden区被new（例外：直接new了一个大于Eden区的对象）
- **绝大部分**Java对象都再新生代被销毁了

#### 对象的分配过程

> 重点！

一个对象被分配，有以下流程：

1. `new`对象
2. 先放到`Eden`区（优先Eden区的TLAB）
   - `Eden`区已满 →  3
   - `Eden`区未满：直接放入
3. 触发**`YGC/Minor GC`**，清除游离的对象（会对`Survivor`与`Eden`都进行清理）
4. 将剩余的对象转至`to`区，并将`from`区内依然存在的对象也放入`to`区，对象每经历一次`YGC`，就会将一个属性值`+1`
5. 若此属性值，大于设置的**阈值**（默认是15），就会`promotion`（晋升）进入老年代

流程如图：

![对象内存分配流程图](http://img.yesmylord.cn//image-20210719000049373.png)

注意：

- `YGC`会清理`Eden`与`Survivor`的游离对象；但是触发`YGC`的只能是`Eden`区满
- `Survivor`区满，会直接将其对象`promotion`至老年代
- `from`区与`to`区是根据`survivor 0`与`survivor 1`区谁满谁不满而言的，空的就是`to`区，另一个就是`from`区
- 如果`new`的对象整个`Eden`都放不下，会去往老年区分配内存；如果老年区放不下，会进行`FGC`，执行完依然放不下，就会出现OOM
- 阈值默认为15，可以通过`-XX:MaxTenuringThreshold=<N>`进行设置
- GC会**频繁**发生在新生代，**很少**发生在老年代，**几乎不在**永久区/元空间收集

另外还有一个拼音问题：

**阈值 （yu zhi）**，阀值就是阈值的错写而已，并不存在阀值这个词（我还念了十几年 fa zhi，没文化，真可怕 >_< ）

#### Minor GC、Major GC、Full GC区分

> 分类：

HotSpot VM的实现，把GC分为两大种类型：

- **部分收集（Partial GC）**
  - 新生代收集（Minor GC / Young GC）：只对新生代进行垃圾收集
  - 老年代收集（Major GC / Old GC）：只对老年代进行垃圾收集
    - 只有**CMS GC**会有只收集老年代的行为
  - 混合收集（Mixed GC）：收集**整个新生代及部分老年代**
    - 只有`G1 GC`会有Mixed GC
- 整堆收集（Full GC）：收集**整个Java堆和方法区**的垃圾收集

> GC触发时机：

- YGC
  - 触发时机：`Eden`区空间不足！
  - 会引发**STW**(Stop the world)，暂停其他用户线程，只有GC结束后，才会使其继续执行
- Major GC
  - 触发时机：老年代空间不足，先触发YGC，如果还不足触发MajorGC
  - STW的时间更长
- Full GC
  - 触发时机：
    - 调用`System.gc()`，系统建议执行Full GC，但不一定
    - 老年代空间不足
    - 方法区空间不足
    - YGC后，进入老年代的平均大小大于老年代可用内存
    - `Eden`、`from`区向`to`区复制时，大小大于`to`区，也大于了老年代内存
  - Full GC应尽量避免

#### 堆空间分代思想

> 为什么要把堆进行分代？不分代不能正常工作吗？

答：不分代也可以正常工作，只不过性能没有分代强。在Java程序中，70%~80%对象都是临时对象，如果不进行分代，每次进行GC都需要遍历很多很多对象，这样性能肯定不会强。如果分为新生代、老年代，就可以大大加快效率

#### 对象提升规则

针对不同年龄段的对象分配原则如下：

- 优先分配到Eden
- 大对象直接分配到老年代
- 长期存活的对象分配到老年代
- **动态对象年龄判断**
  - 如果`Survivor`区中**相同年龄**的所有对象大小的总和大于`Survivor`空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无需等到`MaxTenuringThreshold`要求的年龄
- **空间分配担保**
  - 将`Survivor`区无法存放的对象放入老年代
  - `-XX:HandlePromotionFailure`

#### TLAB

> TLAB（Thread Local Allocation Buffer）

- 堆是线程共享的区域，TLAB是堆上属于线程私有的区域
- `TLAB`在`Eden`区，仅占`Eden`空间的**%1**，我们可以通过选项`-XX:TLABWasteTargetPercent`来设置TLAB空间所占的大小
- JVM首选`TLAB`进行分配，如果内存不够大，会使用**锁方式**确保原子性，在非`TLAB`的`Eden`区域进行分配

> 为什么使用TLAB

​		为避免多个线程操作同一地址，需要使用锁机制，影响分配速度。加入TLAB可以直接避免线程安全问题，提高内存分配效率（这种分配方式也叫**快速分配策略**）

#### 参数设置总结

```
-XX:+PrintFlagsInitial	查看所有的参数的默认初始值
-XX:+PrintFlagsFinal	查看所有的参数的最终值
-Xms	初始堆空间内存（默认为物理内存的1/64）
-Xmx	最大堆空间内存（默认为物理内存的1/4）
-Xmn	设置新生代大小
-XX:NewRatio	设置新生代与老年代的比例
-XX:SurvivorRatio	设置新生代Eden和s0/s1的空间比例
-XX:MaxTenuringThreshold	设置新生代垃圾的最大年龄
-XX:+PrintGCDetails	输出详细的GC处理日志
-XX:HandlePromotionFailure	是否设置空间分配担保（JDK7+失效）
```

`YGC`之前，`JVM`会检查**老年代最大可用连续空间是否大于新生代所有对象的总空间**

- 如果大于：YGC会安全执行
- 如果小于：会查看`HandlePromotionFailure`设置的值
  - 设置值为`true`，那么会继续检查**老年代最大可用连续空间是否大于历次晋升到新生代的对象的平均大小**
    - 若大于：执行一次`YGC`，但是是有风险的
    - 小于，`Full GC`
  - 设置为`false`，执行`Full GC`

JDK7+之后，`HandlePromotionFailure`失效，改为

老年代最大可用连续空间是否大于新生代所有对象的总空间

- 大于：`YGC`
- 小于：`Full GC`

#### 逃逸分析

> 堆是对象分配的唯一选择吗？

不是的，**栈上分配，标量替换技术**都会导致对象不在堆中分配；`TaoBaoVM`创新的**GCIH**实现**off-heap**，可以将生命周期较长的对象放在堆外，提高GC回收效率（**但是，这些技术都没有被采用，所以在Java中，堆就是对象分配的唯一选择！！**）

>逃逸分析（Escape Analysis）：
>
>一种可以有效减少Java程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法

逃逸分析的基本行为就是分析对象动态作用域：

- 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸
- 当一个对象在方法中被定义后，被外部方法所引用，则认为发生逃逸
  - 例如：给成员变量赋值、方法返回值、实例引用传递变量都会发生逃逸

**没有发生逃逸的对象，就可以分配到栈上**，随着方法执行结束，栈空间就被移除，就不需要GC了

> 编译器会对代码做如下优化：

1. **栈上分配**：如果对象没有逃逸，就可以优化为栈上分配

2. **同步省略**：如果一个对象被发现只有一个线程访问，则这个对象可以不考虑同步

   - 在动态编译同步块的时候，JIT编译器可以借助**逃逸分析**来**判断同步块所使用的锁对象是否只能够被一个线程访问**而没有被发布到其他线程
   - 如果没有，那么JIT编译器在编译这个同步块的时候就会**取消对这部分代码的同步**
   - 取消同步的过程就叫**同步省略**，也叫**锁消除**。

   例如如下代码：

   ```java
   public void f(){
       Object hollis = new Object();
       //其实这个锁加了并没用，但不妨碍做一个例子
       synchronized(hollis) {
           System.out.println(hollis);
       }
   }
   ```

   多个线程执行这个代码时，实际上JIT会对这段代码进行优化，最终如下执行

   ```java
   public void f(){
       Object hollis = new Object();
       //同步省略，锁消除
       System.out.println(hollis);
   }
   ```

3. **标量替换**：

   - **标量（Scalar）**：指一个无法再分解成更小数据的数据（Java中原始数据类型就是标量）
   - **聚合量（Aggregate）**：可以分解的数据
   - 经过逃逸分析，如果一个对象不会被外界访问到，就会把这个对象拆解成若干个成员变量（即将聚合量变为标量）

   例如如下代码：

   ```java
   public static void main(String[] args){
   	alloc();
   }
   private static void alloc (){
   	Point point =new Point (1,2) ;
   	System.out.println ("point.x="+point.x+";point.y="+point.y);
   } 
   class Point{
   	private int x;
       private int y;
   }
   ```

   会被优化为：

   ```java
   private static void alloc (){
   	int x = 1;
   	int y = 2;
   	system.out.println("point.x=" + x + "; point.y="+y);
   }
   ```

逃逸分析技术并不成熟：

​		其根本原因就是**无法保证逃逸分析的性能消耗一定能高于他的消耗**。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程。

​		所以呢**HotSpot VM没有采用**栈上分配这种方式，所以**在Java中所有对象都是在栈上分配的**。

----

（具体Java对象会不会栈上分配，这有一些争议。很多博客都写着，因为new 100w个user，开启逃逸分析后，堆中只会存放30w，少了很多，就认为对象可以在栈上存放，但是这些只能说明标量替换的效果，并不能证明真正实现了栈上分配；我查阅多个较为靠谱的资料，可以总结以下内容，如果有错误，请指正！）

结论如下：

> **Java对象一定都在堆内存放；Hotspot VM实际没有实现栈上分配，而是使用标量替换来进行优化。**

（此处结论参考的文献：[美团技术团队此篇博客](https://tech.meituan.com/2020/10/22/java-jit-practice-in-meituan.html)、[尚硅谷宋红康JVM教程](https://tech.meituan.com/2020/10/22/java-jit-practice-in-meituan.html)、[极客时间杨晓峰Java核心技术面试精讲](https://time.geekbang.org/column/intro/100006701)）

> **你也可以认为，除了堆，栈上也存放了对象；因为即使是标量替换，也是将原本的对象拆开了存了进去。**

### Method Areas (Metaspace)

> Jdk7及之前，都叫做方法区；Jdk8之后改为元空间；本节以JDK8+称元空间为标准

#### 栈、堆、方法区的交互关系

线程私有：

- 虚拟机栈
- 本地方法栈
- 程序计数器

线程共享：

- 堆
- **元空间**

如图：

可见**方法区存放着类型的信息**

![关系图](http://img.yesmylord.cn//image-20210720115413117.png)

#### 特点

- 逻辑上属于堆，但其实是**独立于Java堆的内存空间（No-Heap）**
- 属于**共享区域**
- 物理上内存空间可以不连续
- 大小可以固定也可以动态扩展；决定了系统可以保存多少个类
- 如果类太多，元空间存不下，那么会出现OOM
- 与JVM同生死

#### 方法区的演进过程

JDK7及之前，称方法区为**永久代**（方法区和永久代并不等价，仅在hotspot vm实现而言，**两者等价**）

JDK8+，使用**元空间取代了永久代**，但有区别

- 元空间不在虚拟机设置的内存中，而是**直接使用本地内存**
- 内部结构也进行了调整

![演进过程](http://img.yesmylord.cn//image-20210720121810359.png)

#### 方法区设置

> JDK7-

- 使用`-XX:PermSize`来设置永久代初始分配空间，默认值 20.75M
- 使用`-XX:MaxPermSize`来设置永久代最大可分配空间，32位机器默认64M，64位机器默认82M

> JDK8+

- 使用`-XX:MetaspaceSize`与`-XX:MaxMetaspaceSize`取代上述原有两个参数
- 默认值依赖于平台
  - windows下，默认初始值为21M，最大值默认为-1，意味着没有限制

注意：

- 大小可以固定，也可以动态增长
- 如果运行中，超过了默认值21M，那么会触发`Full GC`，并重新设置`MetaspaceSize`
- 为防止频繁的发生Full gc，建议设置大一点

#### 方法区内部结构

> 方法区存放：

- 类型信息
- **运行时常量池**
- 静态变量
- JIT代码缓存
- 域信息
- 方法信息

---

1. 类型信息
   - 对于类、接口、枚举、注解，JVM必须存储以下信息：
     - 全限定类名
     - 直接父类的全限定类名（除了`interface`和`java.lang.Object`，因为他们没有父类）
     - 类的修饰符（例如`pulic abstact final`等）
     - 直接接口的一个有序列表

2. 域信息
   - 存放属性的**相关信息**以及**声明顺序**
     - 域名称
     - 域类型
     - 域修饰符
3. 方法信息
   - 存放方法的信息以及声明顺序
     - 方法名称
     - 方法返回类型（或void）
     - 方法的参数的数量和类型
     - 方法修饰符
     - 方法字节码、操作数栈、局部遍历表及大小（`abstract`和`native`方法除外）
     - 异常表（`abstract`和`native`方法除外）：每个异常处理开始的位置、结束位置、代码处理在PC中的偏移地址、被捕获的异常类的常量池索引
   - 类加载器的信息（哪一个加载器加载的此方法）
4. 类变量
   - 非`final`类变量：
     - 随着类的加载而加载
   - `final`类变量
     - 编译时期就被分配
5. 运行时常量池（**Run-time Constant Pool**）
   - 重点来谈

##### 运行时常量池

要知道运行时常量池，先来搞懂常量池

> 常量池

一个类编译成字节码文件后，仍然需要数据支持，这种数据很大，不能直接存到字节码内，就存放到了**常量池**中

常量池存放的数据有：数量值、字符串值、类引用、字段引用、方法引用

类似这样：

![常量池存储内容](http://img.yesmylord.cn//image-20210720195344921.png)

反编译后常量池内容：

```
Constant pool:
   #1 = Methodref          #5.#24         // java/lang/Object."<init>":()V
   #2 = Fieldref           #4.#25         // User.id:Ljava/lang/String;
   #3 = Fieldref           #4.#26         // User.name:Ljava/lang/String;
   #4 = Class              #27            // User
   #5 = Class              #28            // java/lang/Object
   #6 = Utf8               id
   ...等等...
```

​		常量池可以看做一张表，**存放编译期产生的字面量与符号引用**，虚拟机指令根据这个表查找要执行的类名、方法名、参数类型、字面量等类型。

> 运行时常量池

- **方法区的一部分**
- 编译后字节码文件中的**常量池会在类加载后放到此处**
- 每一个已加载的类型（类或接口）都维护一个常量池
- 运行时常量池中不仅包括编译器的字面量，还有**运行期解析后的方法或字段引用**，此时不再是符号引用，而是真实的地址
- 具有**动态性**：比常量池内容要多一些

#### 方法区演进细节

| 版本    | 方法区细节                                                   |
| ------- | ------------------------------------------------------------ |
| JDK1.6- | 有永久代，**静态变量存放在永久代**                           |
| JDK1.7  | 有永久代，但已经逐步去除永久代；**字符串常量池、静态变量保存在堆中** |
| JDK1.8+ | 无永久代，类型信息、字段、方法、常量保存在本地内存的元空间中，但字符串常量池、静态变量仍然保存在堆中 |

![JVM方法区变化图](http://img.yesmylord.cn//image-20210720210910082.png)

> 为什么要用元空间替换方法区？

1. 为永久代设定空间大小很难确定
   - 某些场景下，使用动态加载类过多，容易导致永久代存满，触发多次`Full GC`，甚至很容易导致OOM
2. 对永久代进行调优十分困难
3. 历史原因：当时Hotspot VM要与JRockit VM进行合并，然而JRokit根本没有永久代（只有HotSpot有永久代）

> 为什么要将`StringTable`（字符串常量池）放在堆中

1. 开发中`StringTable`使用频率很高，但是原本存放在永久代，只有`Full GC`才能清理回收这一块内存，而放在堆中，可以提高回收效率

#### 方法区垃圾回收

方法区的垃圾回收主要回收两部分内容：

1. 常量池中废弃的常量

2. 不再使用的类型

> 1. 常量池中的两大类常量：字面量与符号引用

- 字面量：例如文本字符串、`final`声明的常量值等
- 符号引用
  - 类与接口的全限定名
  - 字段的名称与描述符
  - 方法的名称与描述符

回收策略：只要常量没有被任何地方引用，就会被回收

> 2. 不再使用的类型（类的回收条件十分苛刻）

判断一个类型不再使用，需要同时满足以下三个条件：

- 该类没有实例，该对象的子类也没有任何实例
- 加载该类的类加载器已经被回收（很难达成）
- 该类对应的`java.lang.Class`对象没有在任何地方被引用，无法在任何地方使用反射访问该类的方法

注意：满足上述条件后也只是**被允许**被回收，JVM还提供了一个参数进行控制

`-Xnoclassgc`控制是否回收类

`-verbose:class`、`-XX:+TraceClass-Loading`、`-XX:+TraceClassUnLoading`查看类加载、类卸载信息

> 方法区回收的意义：

在大量使用反射、动态代理、CGLIB等字节码框架，动态生成JSP以及OSGi这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载能力，以保证不会对方法区造成太大压力

## Native Method Interface 本地方法接口

![JVM总结构略图](http://img.yesmylord.cn//image-20210713130642171.png)

本地方法接口，就是**为了给Java程序调用非Java代码**所提供的一个部分。

> 本地方法：Java用`native`关键字修饰，意思是此方法实现不是用Java实现的，可能是使用C/C++实现的等等

例如`Obejct`的`getClass`方法

```java
public final native Class<?> getClass();
// 没有方法体？ 有方法体，只不过是其他语言
```

> 为什么要用本地方法？

JVM要和硬件、操作系统、外界交互，Java语言本身运行速度的并不快，此时用C/C++语言来实现，可以大大提高效率。

## Execution Engine 执行引擎

### 概述

主要任务：

- 负责**将字节码指令解释/编译为对应平台上的本地机器指令**
- 解释执行
- 编译执行

注意：

- 此时的编译称为**后端编译**，与**前端编译**（将java程序编译成字节码文件）不同
- 执行引擎执行的指令由PC决定

> 为什么Java是半编译半解释性语言？

Java一开始只是解释型语言，只可以通过解释器进行解释执行，但是后来JIT的引入，使得Java可以编译执行。

解释可以使Java跨平台、编译可以使Java运行更高效。

### 解释器

> 解释器：根据预定义的规范，对字节码采用**逐行解释**的方式执行，目的是**将字节码文件中的内容翻译**为对应平台的本地**机器指令**执行

解释器有两种：

- 古老的**字节码解释器**
- **模板解释器**

在Hotspot Vm中，解释器有Intercepter模块与Code模块构成

- Intercepter模块：实现了解释器的核心功能
- Code模块：用于管理HotSpot VM在运行时生成的本地机器指令

### JIT编译器

> 编译器：将源代码直接编译成和本地机器平台相关的机器语言
>
> JIT：Just In Time 即时编译技术，可以识别出热点代码，直接将其编译，提高执行效率

Java中的“编译”：

- `.java -> .class`：前端编译器（Javac）
- `.class -> 机器码`：后端运行期编译器（JIT编译器）（Hotspot VM的C1、C2编译器）
- `.java -> 机器码`：静态提前编译器（AOT编译器，**程序运行之前**就进行编译，可能是java未来的发展方向）

> 如何识别出热点代码？

​		一个被多次调用，或者是循环次数较多的循环体都可以叫**热点代码**，对于这个度量标准，有一个阈值以便于分析出真正的热点代码。（热点探测功能）

Hotspot VM会为每一个方法都建立2个不同类型的计数器，分别称为**方法调用计数器（Invocation Counter）**和**回边计数器（Back Edge Counter）**：

- 方法调用计数器：用于统计方法的调用次数
- 回边计数器：用于统计循环体执行的循环次数

在Server模式下默认值为10000次，超过这个值就会进行JIT编译

---

**热度衰减**：

如果不进行热度衰减，随着时间的推移，所有的代码都有可能执行超过了阈值

所以方法调用计数器统计的是一段时间之内的调用次数，如果超过这个时间，值会减少一半（类似于半衰期）

---

我们可以设置Java的执行方式：

- 完全解释：`-Xint`
- 完全编译：`-Xcomp`
- 解释器+即时编译混合模式：`-Xmixed`（默认）

JVM内嵌有两个JIT编译器，分别是`Client Compiler`和`Server Compiler`，通常称为C1与C2编译器，我们可以选择使用

- `-client`：运行在Client模式下且使用C1编译器
  - C1编译器会对字节码进行简单可靠的优化，耗时短
- `-server`（64位系统默认设置，64位系统即使设置client也会被忽略）：运行在Server模式下，使用C2编译器
  - C2编译器会进行耗时较长的优化，以及激进优化，但优化后的代码执行速度更快

`-server`模式下，默认开启**分层编译**策略

> 分层编译：
>
> ​		程序解释执行（不开启性能监控）可以触发c1编译，将字节码编译成机器码，可以进行简单优化，也可以加上性能监控，C2编译会根据性能监控信息进行激进优化。

---

C1与C2的优化策略也有不同：（了解）

C1编译器上主要有方法内联，去虚拟化、冗余消除。

- 方法内联：将引用的函数代码编译到引用点处，这样可以减少栈帧的生成，减少参数传递以及跳转过程
- 去虚拟化：对唯一的实现类进行内联
- 冗余消除：在运行期间把一些不会执行的代码折叠掉

C2的优化主要是在全局层面，逃逸分析是优化的基础。基于逃逸分析在c2上有如下几种优化:

- 标量替换：用标量值代替聚合对象的属性值
- 栈上分配：对于未逃逸的对象分配对象在栈而不是堆
- 同步消除：清除同步操作，通常指`synchronized`

---

未来（了解）：

- JDK10，加入了全新的即时编译器——`Graal`编译器
- JDK 9 引入了AOT编译器（静态提前编译器）

## 直接内存

### 特点

1. 属于OS直接提供的内存

2. 来源于NIO，通过存在堆中的`DirectByteBuffer`操作`Native`内存

   > NIO：New IO / Non-Blocking IO
   >
   > 相对于旧的IO：如 `byte[] / char[]`或是`Stream`
   >
   > NIO：`Buffer`、`Channel`，基于此还有框架`Netty`更快更高效

3. 读写性能更高
4. 直接内存**可能会出现OOM**
5. 分配回收成本较高
6. 不受JVM内存回收管理
7. 可以使用`MaxDirectMemorySize`进行设置；不指定，默认与堆的最大值`-Xmx`参数值一致

这有一个小的Demo，可以体验一下使用ByteBuffer开辟直接内存

```java
public class NativeMemory {
    private static final int Buffer = 1024 * 1024 * 1024; // 1GB内存
    public static void main(String[] args) {
        ByteBuffer byteBuffer = ByteBuffer.allocate(Buffer); // 使用直接内存
        System.out.println("直接内存访问完毕");
        Scanner scanner = new Scanner(System.in);
        scanner.next(); // 暂停一下程序，等待输入
        System.out.println("直接内存开始释放");
        byteBuffer = null;
        System.gc();
        scanner.next(); // 暂停一下程序，等待输入
    }
}
```



## 对象的实例化

在堆中，我们已经讲到过对象分配过程，但是那只是一部分。

### 创建对象的方式

1. `new`创建，包括最基本的`new`，和使用静态方法或者是静态工厂生成的对象。
2. Class的`newInstance()`方法：反射的方式，只能调用空参的构造器，权限必须是`public`（JDK8之后，这个方法过时了）
3. Constructor的`newInstance(Xxx)`：可以调用空参、带参的构造器，权限没有要求
4. 使用`clone()`：不调用任何构造器，但是当前类需要实现`Cloneable`接口，实现`clone()`方法
5. 使用反序列化
6. 第三方库`Objenesis`

### 对象创建的步骤

1. 判断对象的类是否已经**加载、链接、初始化**

   JVM遇到new指令，会去`metaspace`的常量池中定位到一个**符号引用**，检查这个符号引用对应的类是否已经被加载

   如果没有加载，在双亲委派模式下，使用当前类加载器以`ClassLoader + 包名 + 类名`为key，查找对应的`.class`文件，如果没有找到，抛出`ClassNotFoundException`异常；如果找到，进行类加载，生成对应的Class类对象

2. 为对象分配内存

   首先计算对象的大小，然后在堆中划分内存，此时会遇到一个问题：

   - 内存规整：JVM使用**指针碰撞算法（Bump The Pointer）**
   - 内存不规整：JVM使用**空闲列表（Free List）分配**

   > 指针碰撞算法（Bump The Pointer）：
   >
   > 即在内存中，分开已使用的内存和未使用的内存，然后维护一个指针作为分界点；给对象分配内存时，只需要将指针移动相应大小即可（如果GC选择的是`Serial`、`ParNew`这种基于压缩算法的，虚拟机将采用这种分配方式，一般使用带有`compact`过程的收集器时使用指针碰撞）

   > 空闲列表（Free List）：
   >
   > 一个记录哪些内存块可用的列表。分配内存时，从列表找一块足够大的空间划分给对象即可

3. 处理并发安全问题

   - 采用**CAS失败重试**、**区域加锁**保证更新原子性
   - 每个线程先分配一块**TLAB**

4. 初始化分配到的空间

   - 所有属性**设置默认值**，保持对象实例字段在不赋值时就可以直接使用

5. 设置对象的**对象头**

   > 对象头：（下面会进行主要讲解）
   >
   > 包含两部分：运行时数据及类型指针

6. 执行`init`方法进行初始化

   此时才会执行类的构造器方法

### 对象的内存布局

对象在内存中存储，主要分为以下几个部分：

1. 对象头：包含两部分（对象头占12字节，其中markword占8字节，类型指针占4字节）

   - **MarkWord**：存放三大部分即锁信息、GC信息、Hashcode
     - 哈希值
     - GC分代年龄
     - 锁状态标志
     - 线程持有的锁
     - 偏向线程ID
     - 偏向时间戳
   - **类型指针**：指向类元数据`InstanceKlass`，确定该对象所属的类型

（数组的话还要记录数组的长度）

2. 实例数据（Instance Data）

   - 对象真正存储的有效信息（代码中定义的各种字段，包括从父类继承的）
   - 有几个规则：
     - 相同宽度的字段总是被分配到一起
     - 父类中定义的变量会出现在子类之前
     - 如果`CompactFields`参数为`true`，那么子类的窄变量有可能插入到父类变量的空隙

3. 对齐填充（使用占位符保证内存对齐，没什么其他作用，就是为了对齐，可能会存在）

   为什么要对齐？因为CPU读取内存，必须是一整块一整块的读，所以要将内存对齐

![内存中的对象](http://img.yesmylord.cn//image-20210722194926407.png)

如下代码，会占用内存几个字节？
```java
Object o = new Object();
```
分为三部分：
- 对象头：
  - markword：占8字节
  - 类型指针klass point：占4字节
- 实例数据
  - 无实例数据，不占大小
- 字节填充
  - 64位机器的话，必须可以整除8，所以还需添加4字节

一共占用16字节

### 对象访问定位

对象访问有两种访问方式：区别就是**指向对象类型数据的指针**存放在什么位置

- 句柄访问：放在堆中的句柄池
- 直接指针（HotSpot VM默认采用）：放在对象实例数据内

#### 句柄访问

优点：如果对象位置发生移动，也不需要修改到对象类型数据指针的值

缺点：需要在堆中开辟专属空间

![句柄访问](http://img.yesmylord.cn//image-20210722215835229.png)

#### 直接指针

优点：不需要开辟额外空间

![直接指针](http://img.yesmylord.cn//image-20210722220547142.png)

## StringTable

String是使用最广泛的类之一，字符串常量池这个结构存在于堆中，但是比较重要，所以单独进行讲述

### String类

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    private final char value[]; // JDK 1.8
    private final byte value[]; // JDK 1.9
    ...
}
```

- `final`修饰的类不可继承
- 实现`Serializable`，支持序列化；实现了`Comparable`，可以比较大小
- JDK8及以前使用`char[]`存储数据，JDK9+改用使用`byte[]`
- 不管是`char[]`还是`byte[]`均使用final修饰，均不可改变

> 为什么改成使用`byte[]`

因为研究发现，在堆中存储的String对象**大部分**是拉丁字符，如果使用`char[]`做存储，每一个字符需要使用两个字节，这样会导致一半的空间被浪费，所以使用了**`byte[]`数组 +编码标记（encoding-flag）**；对于其他文字（UTF-16）依然使用两个字节去存储

> 不可变的字符序列

- 当对一个字符串重新赋值时，需要**重新指定内存区域**赋值，不能使用原有的`value`进行赋值
- 当对现有的字符串进行连接操作时，也需要**重新指定内存区域**赋值
- 调用`replace`方法修改指定字符或字符串时，需要**重新指定内存区域**赋值，不能使用原有的`value`进行赋值

- 直接使用字面量进行赋值，此时的字符串位于字符串常量池中

demo如下，简单，不做赘述

```java
String s1 = new String("nihao"); // 使用new进行创建
String s2 = "nihao"; // 使用字面量进行创建
String s3 = "nihao";
// == 比较地址
System.out.println(s1 == s2); //false 说明new重新给了地址
System.out.println(s2 == s3); //true 相同字面量来自字符串常量池，所以相同
```

- 字符串常量池中不会存储相同的内容

### String内存分配

Java为8种基本类型和String类型都提供了**常量池**

常量池相当于是一个Java系统级别的缓存

> 字符串常量池String Pool：
>
> 是一个固定大小的`HashTable`（数组+链表），默认值大小长度为1009（JDK6为此值，而且可以自己任意设置；JDK7默认值60013，值可任意；JDK8开始，1009是可以设置的最小值）
>
> - 如果存放的String太多，会导致hash碰撞严重，降低查询效率
> - 常量池**不会存放相同的元素**

基本类型的常量池由系统协调，`String`的常量池比较特殊，使用方法有两种：

- 直接使用字面量复制的对象会存放在常量池中
- 调用`intern()`方法，也会将`String`对象放在常量池中
  - 这是一个`native`方法

demo：存到常量池的两种方式

```java
String s1 = "hello";
String s2 = new String("hello");
System.out.println(s1 == s2); // false
String s3 = s2.intern();
System.out.println(s1 == s2); // false intern方法不会将原有的字符串值改变，而是返回一个新的字符串
System.out.println(s1 == s3); // true
```

---

另外，字符串常量池的位置不同版本也有所区别：

![字符串常量池的位置](http://img.yesmylord.cn//image-20210720210910082.png)

- 1.6之前，存放在方法区的运行时常量池
- 1.7之后移动到了堆内

（插一句，网上很多人说1.8放在了元空间内，简直误人子弟，建议大家找可信的大牛的博客看）

> 为什么移动到了堆中？

原本存放在方法区中，缺点有两个：

1. 永久区内存小
2. 永久区满后，需要`Full GC`，频率低而且回收时间长

### String的拼接操作

结论：

- 常量与常量的拼接结果放在常量池，原理是**编译期优化**
- 只要其中有一个是变量，结果就在堆（非常量池的堆）中，变量拼接的原理是**`StringBuilder`**
- 如果拼接的结果使用`intern()`方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址

demo：

```java
String s1 = "ab";
String s2 = s1 + "c";
String s3 = "abc";
String s5 = "ab" + "c";

System.out.println(s2 == s3);// false 拼接双方只要有一个变量，结果就在非常量池的堆中
System.out.println(s5 == s3); // true 编译器做了优化，检测到此处可以直接优化为"abc"
```

字符串的拼接，如果**其中有一个是变量**，那么会调用`StringBuilder`来进行拼接，下面是上面demo的字节码，注意观察 3 行开始处new了`StringBuilder`，在11行调用了`append()`方法，19行调用了`toString`方法

```java
 0 ldc #9 <ab> //直接从字符串常量池调用
 2 astore_0    // 存放到局部遍历表0的位置（这里的方法是一个static方法，没有this）
 3 new #10 <java/lang/StringBuilder> // 此处new了一个StringBuilder
 6 dup
 7 invokespecial #11 <java/lang/StringBuilder.<init> : ()V>
10 aload_0
11 invokevirtual #12 <java/lang/StringBuilder.append : (Ljava/lang/String;)Ljava/lang/StringBuilder;>
14 ldc #13 <c>
16 invokevirtual #12 <java/lang/StringBuilder.append : (Ljava/lang/String;)Ljava/lang/StringBuilder;>
19 invokevirtual #14 <java/lang/StringBuilder.toString : ()Ljava/lang/String;>
22 astore_1
...
```

上面的操作如果使用StringBuilder来执行，如下：

```java
String s1 = "ab";
String s2 = s1 + "c";
// 执行如下
String s = new StringBuilder();
s.append("ab");
s.append("c");
s2 = s.toString();
```

### new String

`new`一个`String`对象，如下

```java
String s = new String("ab");
```

会创建几个对象？

字节码如下：

可以看到，创建了2个对象：

1. `String`对象

2. 字符串常量池中的`"ab"`对象

```java
 0 new #3 <java/lang/String>
 3 dup
 4 ldc #4 <ab>
 6 invokespecial #5 <java/lang/String.<init> : (Ljava/lang/String;)V>
 9 astore_0
10 return
```

----

进阶：

```java
String s = new String("a")+ new String("b");
```

创建了几个对象？

有五个

1. `StringBuilder`对象
2. `new String("a")`
3. 常量池中的`a`
4. `new String("b")`
5. 常量池中的`b`

其实还有，最后赋值给变量`s`，调用了`StringBuilder`的`toString`方法，这个方法的字节码如下：

```java
 0 new #80 <java/lang/String>
 3 dup
 4 aload_0
 5 getfield #234 <java/lang/StringBuilder.value : [C>
 8 iconst_0
 9 aload_0
10 getfield #233 <java/lang/StringBuilder.count : I>
13 invokespecial #291 <java/lang/String.<init> : ([CII)V>
16 areturn
```

6. `new String("ab")`

总共创建了6个对象！**注意最后一个`ab`并不在常量池中！**

### String的`intern()`方法

```java
public native String intern();
```

执行时，此方法会判断池中是否还有本字符串对象：

- 如果存在，返回池中地址
- 不存在，放入池内，返回地址

仔细观察下面这个demo，进一步体会`itern()`：

```java
String s1 = new String("1");
s1.intern();
String s2 = "1";
System.out.println(s1 == s2);// false
// ----------------我是分割线----------------------
String s3 = new String("1") + new String("1");
s3.intern();
String s4 = "11";
System.out.println(s3 == s4);//JDK1.6 false ; JDK1.7/1.8 true
```

剖析：

上半部分，很好理解：

​		`s1`指向`String`对象原地址，`s2`指向的是来自池中的地址，`s1`调用`intern`方法会返回一个池中的对象地址，但是`s1`并没有接收这个变量，所以结果是`false`

​		但是此处有一个要注意的地方，就是执行`new String("1");`时，已经将`"1"`放入池中，即执行`intern()`方法之前，`"1"`就在池中

下半部分，有点绕：

上一节我们介绍了这个具体的过程，他最后一步调用了StringBuilder的toString方法，但是此时，池中并没有`11`（池中只存在`"1"`）

所以调用`intern`方法后，会将`11`放入池中，此时池中存在的是s3的地址，所以使用s4获取时，获得的也是相同的地址，结果为`true`

> `intern()`方法总结：

- jdk1.6中：
  - 如果串池中有，则不会放入。返回已有的串池中的对象的地址
  - 如果没有，会**把此对象复制一份**，放入串池，并返回串池中的对象地址
- Jdk1.7起：
  - 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
  - 如果没有，则会**把对象的引用地址复制一份**，放入串池，并返回串池中的引用地址

### G1编译器的String的去重操作（了解）

数据得出，堆中存活数据集合内的String对象占`25%`，堆存活数据集合里**重复的String对象**有`13.5%`（注意：这里指的堆是除了池中的堆）

所以为了提高性能，就要对重复的String对象进行去重：

- GC工作时，会访问存活的对象，此时就会检查此对象是否是候选的待去重的`String`对象
  - 如果是：将这个对象引用插入到一个候选队列中。另外一个去重的线程就会处理这个队列。
- 使用一个HashTable来记录所有被String对象引用的不重复的`char`数组。去重时，会查这个hashmap，来看堆上是否存在相同的`char`数组

## Java内存模型JMM

注意，JMM与JVM的结构或者是运行时数据区的组成要分清楚，完全不是一码事

> Java内存模型(即Java Memory Model，简称JMM)本身是一种抽象的概念，并不真实存在
>
> 它描述的是一组规则或规范，通过这组规范**定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式**

JMM要掌握的内容：

- CPU的寄存器、缓存、主存的层次关系
- JVM堆、栈与硬件的对应关系
- 主内存与工作内存
- 共享变量的操作
- JMM如何实现原子性、有序性、可见性
- `happens before`原则
- 从JMM理解`volatile`

### JMM与硬件结构

硬件结构，CPU寄存器是最最最最块的内存了，即使快如主存，在寄存器面前也是不够看，所以引入了缓存

缓存是为了解决主存与寄存器速度不匹配而引入的，但是带来了新的问题，一个变量如果在三个地方都存在，那么他的更新就有延迟，于是JMM有协议规定这方面协同

![图片出处见相关链接](http://img.yesmylord.cn//image-20210817145349764.png)

可见，运行时数据区与硬件结构并没有直接的对应关系。

### 主内存与工作内存

- 所有的变量（包括对象的、方法的、类的变量）都存储在主内存（Main Memory）中。
- 每个线程都有一个私有的本地内存（Local Memory 也叫工作内存），本地内存中存储了该线程以读/写共享变量的拷贝副本。
- **线程对变量的所有操作都必须在本地内存中进行，而不能直接读写主内存。**
- 不同的线程之间无法直接访问对方本地内存中的变量。

### 共享变量

在JMM中，共享变量存放在**主内存**中，每一个线程都可以访问主内存，而且线程内部还有一块**工作内存**，存放共享变量的副本

线程只可以对自己工作内存中的共享变量进行操作，但却不能对主存内的共享变量进行直接的操作

浏览blog过程中发现一张特别容易理解的图：

（通过这个图可以好好想想`volatile`的原理）

![此图出处见相关链接](http://img.yesmylord.cn//image-20210817144328318.png)



### JMM如何实现原子性、可见性、有序性

原子性：操作必须一次性不可中断的完成

可见性：变量修改后，线程是否能马上获取到这个新值

有序性：指令会被进行重新排序，而JMM不能让有序的操作被乱执行

JMM通过`synchronized`、`ReentrantLock`等锁机制保证原子性

通过`volatile`保证可见性和有序性

除此外，还有happens before原则（保证基本内容的顺序）

### happens before原则

1. **程序顺序规则**（线程内必须串行执行）
2. **锁规则**（解锁必须发生在上锁后）
3. **volatile规则**（强迫每次的读写都必须刷新到主内存，不能为了省事直接去工作内存读）
4. **线程启动规则**（线程的`start()`方法先于它的其他操作）
5. **传递性**（A先于B，B先于C，A必先于C）
6. **线程终止规则**（线程的所有操作先于线程的终结）
7. **线程中断规则**
8. **对象终结规则**（构造方法先于`finalize()`方法）

这八个规则确定的内容，**即使没有锁等同步操作，也可以按序执行**



### 从JMM理解`volatile`

`volatile`有两个特性：

- 实现了可见性
- 实现了有序性

那么他是如何实现的呢

#### 实现可见性

`volatile` 强迫对被修饰变量的读写刷新在主存

即如果你要写或是读一个`volatile`变量，必须从主存拿（如果不修饰，线程会偷懒，直接就读取工作内存中存放的变量的副本）

注意：**volatile 只对原子操作有限制**，典型的例子 `i++`就不是一个原子操作，它包括两步，+1与赋值

如何实现？**缓存一致性协议** + **happens before规范**

缓存一致性协议有很多，Intel家的MESI（缓存一致性协议）逻辑如下：

当CPU写数据时，**如果发现操作的变量是共享变量**，会**发出信号通知其他CPU将该变量的缓存行置为无效状态**，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

#### 实现有序性

禁止指令重排序（多核下，为了进一步提高CPU计算速度，引入了流水线，流水线下，就会对指令进行重排序）

如何实现？**内存屏障**

> 内存屏障(`Memory Barrier`)：
>
> 又称内存栅栏，**是一个CPU指令**，它的作用有两个：
>
> - 一是保证特定操作的执行顺序
> - 二是保证某些变量的内存可见性（利用该特性实现`volatile`的内存可见性）
>
> `volatile`就是通过内存屏障实现的

如果在指令间插入一条`Memory Barrier`则会告诉编译器和CPU，不管什么指令都不能和这条`Memory Barrier`指令重排序，也就是说通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化。

`Memory Barrier`的另外一个作用是强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本

![写](http://img.yesmylord.cn//image-20210817155630683.png)

![读](http://img.yesmylord.cn//image-20210817155642070.png)

图片均来自与敖丙博客，见相关链接

## 文章相关链接

1. [尚硅谷JVM教程](https://www.bilibili.com/video/BV1PJ411n7xZ)：强推，最强JVM视频教程
2. [《深入理解Java虚拟机》阅读笔记](https://github.com/TangBean/understanding-the-jvm)：省下看书的时间
3. 《深入理解Java虚拟机》：书还是要看的
4. JMM参考1：[博客](https://blog.csdn.net/guoguo527/article/details/116432225?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162917169316780261986543%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162917169316780261986543&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-116432225.first_rank_v2_pc_rank_v29&utm_term=jmm&spm=1018.2226.3001.4187)
5. JMM参考2：[博客](https://blog.csdn.net/javazejian/article/details/72772461?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162918225216780271546106%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=162918225216780271546106&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~hot_rank-1-72772461.first_rank_v2_pc_rank_v29&utm_term=jmm&spm=1018.2226.3001.4187)

