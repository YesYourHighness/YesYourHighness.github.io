---
title: 设计模式
date: 2022-02-14 20:01:20
tags:
- 设计模式
categories:
- 设计模式

---

<center>
    引言：设计模式（未完）
</center>

<!-- more -->

# 设计模式

## 设计基本原则

1. 封装变化
2. 针对接口编程，而不是针对实现编程
3. 多用组合，少用继承
4. OP原则：对扩展开放，对修改关闭

## OOP

### 面向对象软件开发的三个阶段

1. OOA：面向对象分析
2. OOD：面向对象设计
3. OOP：面向对象编程。一种编程范式或是编程风格。

> 什么是OOP？

一句话：**以类和对象作为基本单元，将封装、继承、抽象、多态四个特性作为代码设计和实现的基石**。

> 如何判断一个语言是否是面向对象的？

只要看这个语言是否实现了四大特性（不一定全都要实现，放宽要求的话，只要具有类、对象这两个概念，这个语言就可以是OO的了）

> 四大特性的意义：

- 封装：
  1. 保证系统运行的**安全性**，防止意外的更改数据
  2. 提高**易用性**（减少不必要的额外操作提高易用性，比方说一个电视机，有很多配置，这需要很大的学习成本，但是如果只有开机、关机、切换频道三个按钮，就很容易上手）
- 抽象：
  1. 关注大逻辑，忽略小细节
- 继承：
  1. 代码复用
- 多态：
  1. 可扩展性
  2. 复用性

> Java如何实现四大特性：

- 封装：通过**访问权限控制**（`private`、`protected`、`default`、`public`）
- 抽象：抽象类与接口类
- 继承：类之间的继承
- 多态：父类可以引用子类对象；重写；重载

### Java抽象类

> Java 抽象类

1. 抽象类**不能被实例化**（如果被实例化，就会报错，编译无法通过）
2. 抽象类中不一定包含抽象方法，但是有抽象方法的类必定是抽象类。
3. 抽象类中的抽象方法只是声明，不包含方法体，就是不给出方法的具体实现也就是方法的具体功能。
4. **构造方法，类方法（用 static 修饰的方法）不能声明为抽象方法**。
5. 抽象类的子类必须给出抽象类中的抽象方法的具体实现，除非该子类也是抽象类。
6. **抽象类可以有具体的方法**

> 如何决定使用接口还是抽象类？

- 如果我们要表示一种 `is-a` 的关系，并且是为了解决**代码复用**的问题，我们就用抽象类
- 如果我们要表示一种 `has-a` 关系，并且是为了解决**抽象而非代码复用**的问题，那我们就可以使用接口

# 创建型

> 创建型：阐述如何优雅的创建一个对象

## 单例模式

### 单例模式的适用场景

> 单例模式：一个类只创建一个实例

- 防止资源的访问冲突
  - 解决资源访问冲突的方法很多：分布式锁、使用并发类、而使用单例模式是最简单的一种解决方法
- 表示**全局唯一**
  - 有时候业务上要表示系统全局唯一的类，比如配置文件

### 单例模式的实现要点

要实现一个单例模式，有很多种方式：

- 将构造方法设为`private`【必选】
- 是否懒加载
- 保证创建时的线程安全（用`volatile`）
- 考虑是否会被反射破坏单例性

### 实现单例模式的5种方式

#### 饥饿式

特点：`private`构造方法、实例是由`private static final`修饰

- 线程安全（类成员，而且有`final`）
- 初始化时就会被加载

```java
public class Hungary {
    private static final Hungary hungary = new Hungary();
    private Hungary(){}
    public Hungary getInstance(){
        return hungary;
    }
}
```

其中初始化时就加载是饥饿式单例一直被诟病的一点

- 缺点：初始化时就加载，延长了启动时间，占用了内存（万一没有使用此单例，相当于白白浪费了内存）

> 但是这几个缺点真的很致命吗？

- 对于延长了启动时间：如果将启动时间延迟到第一次使用时，那么不就延长了第一次使用时的响应时间吗？
- 对于占用了内存：按照fail-fast（有问题及早暴露）这一观点，提早创建，如果内存不够，可以提早出现OOM，便于去修复

所以饥饿式的两个缺点其实也**不是很致命**（所以不该对于饥饿式的态度好像有点谈虎色变的感觉）

#### 懒汉式

特点：将创建延迟到第一次使用时

- 线程安全：为保证线程安全，对`static`方法添加了`synchronized`关键字，相当于给类对象`Lazy.class`加锁
- 第一次使用时才会被加载

```java
public class Lazy {
    private static Lazy lazy;
    private Lazy(){}
    public static synchronized Lazy getInstance(){
        if(lazy == null){
            lazy = new Lazy();
        }
        return lazy;
    }
}
```

懒汉式的缺点很明显，使用`synchronized`这种重量级锁，会大大降低并发性（几乎和串行没有区别）

使用要注意：

- 如果频繁地用到，那频繁加锁、释放锁及并发度低 等问题，会导致性能瓶颈

- 如果使用并不频繁，懒汉式也是一种比较好的实现方式

#### DCL

双重检查懒汉式（Double Check Lazy）

特点：解决懒汉式的并发程度太低的问题

- 线程安全：使用两次`if`保证速度，使用`volatile`保证创建对象安全

```java
public class DCL {
    private volatile static DCL dcl;
    private DCL(){}
    public static DCL getInstance(){
        if(dcl == null){
            synchronized (DCL.class){
                if(dcl == null){
                    dcl = new DCL();
                }
            }
        }
        return dcl;
    }
}
```

> 为什么要加`volatile`？

因为`new`创建一个对象，其实有三个阶段：

1. 分配内存
2. 初始化
3. 指向引用

其中CPU判断2与3并不存在先后执行关系，所以有可能指向引用之后，依然没能初始化，但是对象已经可以拿到了（因为有了引用），所以线程会把它拿回工作内存

因此要加`volatile`，避免重排序

> 为什么要两次判断是否为空？

第二次判断很好理解，为什么要加第一重判断？

如果没有第一重的判断，那么多个线程会为了获取`DCL.class`锁而进入等待，而且也没有办法唤醒

如果有第一重判断，那么会让没拿到锁的线程先去做其他事，提高并发度

#### 静态内部类

静态内部类的特点：外部类被加载，内部类不会被加载

- 懒加载：内部类的特点
- 线程安全：`static final`

```java
public class StaticInnerClass {
    static class InnerClass{
        private static final StaticInnerClass innerClass = new StaticInnerClass();
    }
    private StaticInnerClass(){}
    public StaticInnerClass getInstance(){
        return InnerClass.innerClass;
    }
}
```

#### 枚举

枚举可以做到真正的单例（不会被反射创建新的对象）

```java
public enum SingleEnum {
    SingleEnum;
    public static SingleEnum getInstance(){
        return SingleEnum;
    }
}
```

> 为什么反射创建不了枚举对象？

JDK源码如下：

```java
if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
```

如果发现类型是Enum，就会抛出异常

如果是其他的枚举类，可以通过反射获取构造器，设置许可为true的方式创建对象，如下:

```java
DCL instance1 = DCL.getInstance();
Constructor<DCL> constructor = DCL.class.getDeclaredConstructor();
constructor.setAccessible(true);
DCL instance2 = constructor.newInstance();
System.out.println(instance1); // @28a418fc 哈希值不同，说明创建了两个对象，破坏了单例
System.out.println(instance2); // @5305068a
```

但如果枚举类这么做就会报错

### 不同情况下的单例模式

- **进程内唯一**的单例模式：在多个线程（一个进程的多个线程）内，保证单例（上述的五种单例模式都是进程唯一的单例模式）
- **集群唯一**的单例模式：在多个进程间，保证单例

> 如何在进群内保证一个对象是单例？

1. 需要把共享的对象，**序列化**到外部一个**共享区域**；
2. 使用时上锁，反序列化后使用；
3. 使用完后，并将对象序列化回该区域，释放锁；

### 多例模式

> 所谓多例模式：
>
> - 可以理解为创建的对象的个数是有限的
> - 也可以理解为**同一类**的对象为单例，不同类之间创建的对象不同

1、可以创建的对象个数有限

```java
public class MultiCaseMode1 {
    private static final int size = 3;
    private static final Map<Integer, InetSocketAddress> server = new ConcurrentHashMap<>();
    static {
        server.put(0, new InetSocketAddress("127.0.0.1", 8080));
        server.put(1, new InetSocketAddress("127.0.0.1", 8081));
        server.put(2, new InetSocketAddress("127.0.0.1", 8082));
    }

    private MultiCaseMode(){}

    public static InetSocketAddress getInstance(){
        Random random = new Random();
        return server.get(random.nextInt(size));
    }
}
```

这样每次`getInstance`返回的就是三个对象之一

2、同一类创建的对象个数有限

```java
public class MultiCaseMode2 {
    private static final ConcurrentHashMap<String, Object> instance = new ConcurrentHashMap<>();

    private MultiCaseMode2(){}

    public static Object getInstance(String key){
        instance.putIfAbsent(key, new Object());
        return instance.get(key);
    }
}
```

这样，对于同一类（即相同key）的对象，会获得相同的对象。

## 工厂模式

### 简单工厂模式

		开发中经常使用，如果涉及到两个类之间的转换，比方说要通过A类的大部分成员去生成一个新类B，这时候一般不会在逻辑中进行一堆的`set`操作，而是将生成的逻辑写在A中，直接通过`A.getB(param)`获得B类

### 工厂模式

> 开发中如何确定选择简单工厂模式还是工厂模式？

原则：当**创建逻辑比较复杂**的时候使用工厂模式，其余使用简单工厂模式即可

什么算创建复杂的逻辑？

1. 涉及到对不同的类型，创建了不同的对象（往往有很多个`if-else`嵌套）
2. 创建涉及到很多个对象

### 使用工厂模式实现一个简单的IOC

IOC就是Spring的核心思想之一，作为容器，创建我们所需要的类；

> 工厂模式与IOC的区别是什么？

-  IOC其实就是一个大的工厂类，只不过**工厂类是创建某个类**，而**IOC可以创建整个应用中你想让其创建的类**。
-  简单来说，`IOC = 配置文件 + 工厂模式 + 反射`

---

IOC的核心功能有三个：

- 配置解析
- 对象创建
- 对象生命周期管理

此处实现的IOC只涉及一些简单的逻辑：

1、启动类及两个简单的实体类

```java
public class Main {
    public static void main(String[] args) {
        MyApplicationContext context = new MyClassPathXmlApplicationContext("beans.xml");
        A a = (A) context.getBean("A");
        System.out.println(a);
    }
}

public class A {
    private B b;

    public A() {}

    public A(B b) {
        this.b = b;
    }
    // 省去get、set方法

public class B {
    private String name;
    private Integer age;

    public B() {}

    public B(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
    // 省去get、set方法
}
```

2、`MyApplicationContext`及其实现类

```java
public interface MyApplicationContext {
    Object getBean(String beanId);
}

public class MyClassPathXmlApplicationContext implements MyApplicationContext {

    private MyBeansFactory myBeansFactory;
    private MyBeanConfigParser myBeanConfigParser;

    public MyClassPathXmlApplicationContext(String configLocation) {
        this.myBeansFactory = new MyBeansFactory();
        this.myBeanConfigParser = new MyXmlBeanConfigParser();
        loadBeanDefinitions(configLocation);
    }

    private void loadBeanDefinitions(String configLocation) {
        InputStream in = null;
        try {
            in = this.getClass().getResourceAsStream("/" + configLocation);
            if (in == null) {
                throw new RuntimeException("Can not find config file: " + configLocation);
            }
            List<MyBeanDefinition> beanDefinitions = myBeanConfigParser.parse(in);
            myBeansFactory.addBeanDefinitions(beanDefinitions);
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }


    @Override
    public Object getBean(String beanId) {
        return myBeansFactory.getBean(beanId);
    }
}
```

3、定义类与枚举类等辅助类

```java
public class MyBeanDefinition {
    private String id;
    private String className; // 类的全限定名
    private List<MyConstructorArg> constructorArgList = new ArrayList<>();
    private MyScope scope = MyScope.SINGLETON;
    private boolean lazyInit = false; // 延迟加载
    // 省去get、set方法
}
public class MyConstructorArg {
    private boolean isRef; // 是否是引用其他类
    private Class type;
    private Object arg;
    // 省去get、set方法
}
public enum MyScope {
    SINGLETON, // 表示单例模式
    PROTOTYPE; // 表示原型模式
}
```

4、解析类（此处的解析类某处是设死值，这不是核心，只是为了让前后能跑起来，核心是下面的工厂类，可以跳过）

```java
public interface MyBeanConfigParser {
    List<MyBeanDefinition> parse(InputStream in);
    List<MyBeanDefinition> parse(NodeList configContent);
}
public class MyXmlBeanConfigParser implements MyBeanConfigParser {
    @Override
    public List<MyBeanDefinition> parse(InputStream in) {
        // DOM方式解析xml文件
        DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
        NodeList beanList = null;
        try {
            DocumentBuilder db = dbf.newDocumentBuilder();
            Document doc = db.parse(in);
            beanList = doc.getElementsByTagName("bean");
            System.out.println("该xml文件有：" + beanList.getLength() + "个bean");
        } catch (SAXException | IOException | ParserConfigurationException e) {
            e.printStackTrace();
        }
        return parse(beanList);
    }

    @Override
    public List<MyBeanDefinition> parse(NodeList beanList) {
        if(beanList == null){
            throw new RuntimeException("the resolve result is null");
        }
        List<MyBeanDefinition> beanDefinitions = new ArrayList<>();
        for (int i = 0; i < beanList.getLength(); i++) {
            Node item = beanList.item(i);
            addToList(beanDefinitions, item);
        }
        System.out.println(beanDefinitions);
        return beanDefinitions;
    }

    private void addToList(List<MyBeanDefinition> beanDefList, Node node) {
        NamedNodeMap map = node.getAttributes();
        MyBeanDefinition def = new MyBeanDefinition();

        // 属性设置
        Node id = map.getNamedItem("id");
        Node classPath = map.getNamedItem("class");
        Node scope = map.getNamedItem("scope");
        Node lazyInit = map.getNamedItem("lazy-init");
        if(id == null || classPath == null){
            throw new RuntimeException("bean has no Id or ClassPath");
        }
        def.setId(id.getNodeValue());
        def.setClassName(classPath.getNodeValue());
        if(scope != null){
            String value = scope.getNodeValue();
            if(value.equalsIgnoreCase(MyScope.SINGLETON.name())){
                def.setScope(MyScope.SINGLETON);
            }else if(value.equalsIgnoreCase(MyScope.PROTOTYPE.name())){
                def.setScope(MyScope.PROTOTYPE);
            }else {
                throw new RuntimeException("bean scope has error");
            }
        }
        if(lazyInit != null){
            def.setLazyInit(new Boolean(lazyInit.getNodeValue()));
        }

        // 构造器设置
        NodeList childNodes = node.getChildNodes();
        List<MyConstructorArg> constructorArgList = new ArrayList<>();
        for (int i = 0; i < childNodes.getLength(); i++) {
            Node item = childNodes.item(i);
            if("#text".equals(item.getNodeName()) ){
                // 换行+空格视为一个文本节点
                continue;
            }
            NamedNodeMap childMap = item.getAttributes();
            Node ref = childMap.getNamedItem("ref");
            Node type = childMap.getNamedItem("type");
            Node value = childMap.getNamedItem("value");
            MyConstructorArg myConstructorArg = new MyConstructorArg();
            if(ref != null){
                myConstructorArg.setRef(true);
                myConstructorArg.setType(B.class);
                myConstructorArg.setArg("B");
            }
            if(type != null){
                if("String".equalsIgnoreCase(type.getNodeValue())){
                    myConstructorArg.setType(String.class);
                    myConstructorArg.setArg(value.getNodeValue());
                }
                if("Integer".equalsIgnoreCase(type.getNodeValue())){
                    myConstructorArg.setType(Integer.class);
                    myConstructorArg.setArg(Integer.valueOf(value.getNodeValue()));
                }
            }
            constructorArgList.add(myConstructorArg);
        }
        def.setConstructorArgList(constructorArgList);

        beanDefList.add(def);
    }
}
```

5、工厂类（核心）

```java
public class MyBeansFactory {

    private ConcurrentHashMap<String, Object> singletonObjects = new ConcurrentHashMap<>();
    private ConcurrentHashMap<String, MyBeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>();


    public void addBeanDefinitions(List<MyBeanDefinition> beanDefinitionList) {
        for (MyBeanDefinition definition : beanDefinitionList) {
            this.beanDefinitionMap.putIfAbsent(definition.getId(), definition);
        }
        for (MyBeanDefinition definition : beanDefinitionList) {
            if (!definition.isLazyInit() && definition.getScope().equals(MyScope.SINGLETON)) {
                // 如果非懒加载且不是单例模式
                createBean(definition);
            }
        }
    }


    public Object getBean(String beanId) {
        MyBeanDefinition beanDefinition = beanDefinitionMap.get(beanId);
        if (beanDefinition == null) {
            throw new NoSuchBeanDefinitionException("Bean is not defined: " + beanId);
        }
        return createBean(beanDefinition);
    }

    private Object createBean(MyBeanDefinition definition) {
        if (definition.getScope().equals(MyScope.SINGLETON) && singletonObjects.contains(definition)) {
            // 如果是单例且单例map还有此key
            return singletonObjects.get(definition.getId());
        }
        Object bean = null;
        try {
            Class<?> beanClass = Class.forName(definition.getClassName());
            List<MyConstructorArg> args = definition.getConstructorArgList();
            if (args.isEmpty()) {
                bean = beanClass.newInstance();
            } else {
                Class<?>[] argClass = new Class[args.size()];
                Object[] argObjects = new Object[args.size()];
                for (int i = 0; i < args.size(); i++) {
                    MyConstructorArg arg = args.get(i);
                    if (!arg.isRef()) {
                        // 非引用类型
                        argClass[i] = arg.getType();
                        argObjects[i] = arg.getArg();
                    } else {
                        // 引用类型
                        MyBeanDefinition refBeanDefinition = beanDefinitionMap.get(arg.getArg().toString());
                        if (refBeanDefinition == null) {
                            throw new NoSuchBeanDefinitionException("Bean is not defined");
                        }
                        argClass[i] = Class.forName(refBeanDefinition.getClassName());
                        argObjects[i] = createBean(refBeanDefinition);
                    }
                }
                bean = beanClass.getConstructor(argClass).newInstance(argObjects);
            }
        } catch (ClassNotFoundException | InstantiationException | IllegalAccessException | NoSuchMethodException | InvocationTargetException e) {
            e.printStackTrace();
        }
        if (bean != null && definition.getScope().equals(MyScope.SINGLETON)) {
            singletonObjects.putIfAbsent(definition.getId(), bean);
            return singletonObjects.get(definition.getId());
        }
        return bean;
    }
}
```

## 建造者模式

### 建造者模式

> 建造者模式解决什么问题？

创建一个对象时，往往使用它的构造方法与`set`方法去创建这个对象，但是这样存在几个问题：

1. 如果需要设置的成员很多，那么会导致构造函数的参数很多

   - 导致**代码可读性、易用性变差**
   - 调用时可能会**搞错参数的顺序**

2. 如果参数之间存在依赖关系（比如A参数必须大于B参数等），校验逻辑放在构造函数内也不够优雅

3. 避免对象存在**无效状态**

   ```java
   Rectangle r = new Rectangle();// 创建了长方形对象(无效状态)
   r.setWidth(2); // 设置了宽，但是只有宽的长方形也是不能用的(无效状态)
   r.setHeight(3);// 宽、高都有(有效状态)
   ```

   建造者模式可以避免处于无效状态的对象被别的地方使用导致出错。

例如这个例子，一个资源池：

```java
public class ResourcePoolConfig {
    private static final int DEFAULT_MAX_TOTAL = 8;
    private static final int DEFAULT_MAX_IDLE = 8;
    private static final int DEFAULT_MIN_IDLE = 0;

    /**
     * 四个成员变量： name是必配项，其余三个为可选项
     * 因此一般的设计为构造函数传递name，其余的参数使用set传
     */

    private String name;
    private int maxTotal = DEFAULT_MAX_TOTAL;
    private int maxIdle = DEFAULT_MAX_IDLE;
    private int minIdle = DEFAULT_MIN_IDLE;

    public ResourcePoolConfig(String name) {
        this.name = name;
    }
    // 省去get、set方法
}
```

发现`maxIdle`一定要大于等于`minIdle`，这样参数之间存在逻辑，我们就可以使用建造者模式

```java
public class ResourcePoolConfigV2 {
    private String name;
    private int maxTotal;
    private int maxIdle;
    private int minIdle;

    public ResourcePoolConfigV2(Builder builder) {
        this.name = builder.name;
        this.maxTotal = builder.maxTotal;
        this.maxIdle = builder.maxIdle;
        this.minIdle = builder.minIdle;
    }

    public static class Builder{
        private static final int DEFAULT_MAX_TOTAL = 8;
        private static final int DEFAULT_MAX_IDLE = 8;
        private static final int DEFAULT_MIN_IDLE = 0;

        private String name;
        private int maxTotal = DEFAULT_MAX_TOTAL;
        private int maxIdle = DEFAULT_MAX_IDLE;
        private int minIdle = DEFAULT_MIN_IDLE;

        public Builder(String name) {
            // 校验逻辑
            if("".equals(name)){
                throw new IllegalArgumentException();
            }
            this.name = name;
        }
        // 省去get set方法
    }
}

```

> 建造者模式的缺点

代码重复，发现建造者中有原对象的成员（**代码冗余度高**）

> 建造者模式与工厂模式的区别

生活的例子：顾客走进一家餐馆点餐，我们利用工厂模式，根据用户不同的选择，来制作不同的食物，比如披萨、汉堡、沙拉。对于披萨来说，用户又有各种配料可以定制，比如奶酪、西红柿、起司，我们通过建造者模式根据用户选择的不同配料来制作披萨

- 工厂模式：创建不同但是相关类型的对象
- 建造者模式：创建一种逻辑复杂的对象

> Lombok有注解`@Builder`就是建造者模式的使用

（关于此注解可以看此篇[博客](https://blog.csdn.net/baidu_35085676/article/details/89193416)）

## 原型模式



# 结构型

结构型：总结了一些类或对象组合在一起的**经典结构**

## 代理模式

## 桥接模式

简单的设计模式，实际中并不常用

> 什么是桥接模式

- 抽象和实现解耦：
  - 比如JDBC与Mysql、Oracle具体的Driver实现
  - 也可以理解为log4j规范与Slf4j具体实现的关系
- 组合优于继承

## 装饰者模式

### 模式概念

> 装饰者模式解决的问题：

当一个类的子类很多时，如果我们想给父类扩展一个功能，那么所有的子类都会发生变化（**类爆炸**）

> 什么是装饰者模式？

**动态的将责任附加到对象上**，若要扩展功能，装饰者提供了比继承更有弹性的替代方案

具体实现上，装饰者类会继承同一个类（**其实就是使用多态的特性实现对功能的增强**），然后内部含有一个父类对象的引用（下面的`InputStream`就是例子）

> 装饰者模式与代理模式的区别

- 体现的特性不同：
  - 装饰者模式体现多态性
  - 代理模式体现封装性
- 要实现的目的不同：
  - 装饰者模式是为了**增强功能**
  - 代理模式是为了**实现不属于自己的功能**

### 重要的例子

这里以`InputStream`类来说明：

`InputStream`是一个抽象类，他有几个方法：

```java
public abstract class InputStream implements Closeable {
	public abstract int read() throws IOException;
    // 省略 skip()等方法
    public void close() throws IOException {}
}

```

他有很多个子类，其中有一个子类`FilterInputStream`，该类也有子类`BufferedInputStream`、`DataInputStream`

- `BufferedInputStream`：为流提供了缓冲区
- `DataInputStream`：为流提供了读取基本类型的方法（例如：`readInt`、`readLong`不需要思考具体的读取细节）

这个例子中，`BufferedInputStream`、`DataInputStream`就是**装饰类**，他们继承了`InputStream`，使用的时候可以这么用：

```java
InputStream in = new FileInputStream("/test.txt");
BufferedInputStream bs = new BufferedInputStream(in);
while (bs.read() != -1){
    // ...
}

```

> 为什么此处使用装饰者模式？

		如果我们想实现缓冲区、读取单个基本类型的字节等等这样的功能，如果直接在父类`InputStream`类实现这个功能，那么会导致其所有子类都会有这个功能，这非我本意

> 为什么需要有一个中间类`FilterInputStream`，而不是直接使用`BufferedInputStream`继承`InputStream`？（此处比较难理解）

假设`BufferedInputStream`直接继承了`InputStream`，那么对于`InputStream`的抽象方法（比如说`read`方法），我们需要重写：

```java
public class BufferedInputStream extends InputStream {
    // 假设直接继承了 InputStream
    protected volatile InputStream in;
    protected BufferedInputStream(InputStream in) {
        this.in = in;
    }
    // 下面这个方法需要重写
    public int read(){
        in.read();
    }
    // 其他。。。
}

```

		同理，对于`DataInputStream`，我们也需要这样实现一遍，代码冗余度很高，因此抽出来`FilterInputStream`这个方法

因此如果看JDK源码就是这样：

```java
public class FilterInputStream extends InputStream {

    protected volatile InputStream in;
    
    public int read() throws IOException {
        return in.read();
    }
	// 其他方法
}

```

## 适配器模式

> 将不兼容的接口转换为可兼容的接口

一种补偿措施，补救设计缺陷：

- 有缺陷的接口：参数过多、命名不规范
- 替换依赖的外部系统
- 兼容老版本接口
- 适配不同格式的数据

## 门面模式

> 什么是门面模式

门面模式为子系统提供一组统一的接口，定义一组高层接口让子系统更易用

（我的理解：即**封装+抽象**，封装难用方法，抽象出一个简洁好用的方法）

> 门面模式与适配器模式的区别

- 门面模式：解决**多接口整合的问题**
- 适配器模式：解决**接口过时或无法使用的问题**

> 遇到什么问题可以使用门面模式呢？

**解决性能问题**：

		假设客户端与服务器之间通信，服务器暴露了3个接口A、B、C，某业务需要客户端请求A、B、C三个接口。
	
		但是客户端是APP，App 和服务器之间是通过移动网络通信的，网络通信耗时比较多 
	
		因此我们可以将A、B、C封装在一个D接口中（门面接口），让客户端直接请求门面接口即可，这样就将网络通信的次数减少到1次

---

注意：

- 门面接口如果数量少，可以直接和普通的接口放在一起，如果比较多的门面接口的话，还是专门给门面接口放一个包比较好

## 组合模式

**对于具有树结构的数据有奇效**，比如:

1. 文件与目录
2. 员工与部门

一个目录下可以有目录也可以有文件，常有的操作是统计一个目录包含的文件数量，及一个目录的大小

如果不使用设计模式，我们可能会设计成：

```java
public class FileSystemNode {
    private String path;
    private boolean isFile;
    private List<FileSystemNode> subNodes = new ArrayList<>();
	// 省去一些方法
}

```

而使用组合模式的话：

三个类：文件类与目录类继承文件系统类

```java
public abstract class FileSystemNode {
    protected String path;
    public FileSystemNode(String path) {
        this.path = path;
    }
    public abstract int countNumOfFiles(); // 统计文件数量
    public abstract long countSizeOfFiles(); // 统计文件大小
    public String getPath() {
        return path;
    }
}

```

文件类：

```java
public class File extends FileSystemNode{
    java.io.File file;
    public File(String path) {
        super(path);
        this.file = new java.io.File(path);
    }
    @Override
    public int countNumOfFiles() {
        return 1;
    }
    @Override
    public long countSizeOfFiles() {
        if(!file.exists()){
            return 0;
        }
        return file.length();
    }
}

```

目录类：

```java
public class Directory extends FileSystemNode {
    private List<FileSystemNode> subNodes = new ArrayList<>();
    public Directory(String path) {
        super(path);
    }
    @Override
    public int countNumOfFiles() {
        int num = 0;
        for (FileSystemNode subNode : subNodes) {
            num += subNode.countNumOfFiles();
        }
        return num;
    }
    @Override
    public long countSizeOfFiles() {
        int size = 0;
        for (FileSystemNode subNode : subNodes) {
            size += subNode.countSizeOfFiles();
        }
        return size;
    }
    public void addSubNode(FileSystemNode fileOrDir) {
        subNodes.add(fileOrDir);
    }
    public void removeNode(FileSystemNode fileOrDir) {
        int size = subNodes.size();
        int i = 0;
        for (; i < size; ++i) {
            if (subNodes.get(i).getPath().equalsIgnoreCase(fileOrDir.getPath())) {
                break;
            }
        }
        if (i < size) {
            subNodes.remove(i);
        }
    }
}

```

## 享元模式

> 所谓享元模式，就是共享元数据。
>
> 使用享元模式的目的只有一个，就是**为了节省内存**

比如有一个**在线象棋项目**，每一个房间都有一个牌局，有几十万个房间，每个房间都有一副象棋。

一个象棋类，需要记录 颜色、位置、棋的种类

但其实，对于每一个房间来说，只有棋的位置是不同的，其他元素都是相同的，因此对于除位置外的元素我们都可以抽为一个类，使用享元模式进行共享，节约内存

```java
public class ChessPieceUnit {
    // 棋子样式
    private int id;
    private String text;
    private Color color;
}
public class ChessPiece {
    // 棋子类
    private ChessPieceUnit chessPieceUnit;
    private int positionX;
    private int positionY;
}

```

> Java中的享元模式

比如Integer的整型池、字符串常量池都属于享元模式的实现

> 享元模式与单例模式、缓存、对象池的区别

- 享元模式与单例模式区别
  - 享元模式：一个类可以有多个对象（类似于多例模式，但是目的是为了节省内存，而不是限制对象个数）
  - 单例模式：一个类只能创建一个对象
- 享元模式与缓存与对象池的区别
  - 享元模式：为了节省内存
  - 缓存：为了提高访问效率
  - 对象池：为了节省对象的创建时间

# 行为型

## 观察者模式

### 概念介绍

观察者模式（即pub/sub模式）

- pub出版者即主题Subject
- sub订阅者改称为观察者Observe

**观察者模式=主题+观察者**

> 观察者模式：
>
> 		定义了对象之间的**一对多依赖**（`主题:观察者=1:n`），这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新

### Java中的观察者模式

Java自带了观察者模式（`Observe`接口（sub）与`Observable`类（pub）），但是**存在一些问题**

- `Observable`是**一个类**，这意味着在单继承的Java中，使用不是很方便
- `Observable`的API中，`setChange()`是`protected`修饰的，**违反了多用组合、少用继承的原则**

因此**掌握其设计思想才是重要的**！

```java
public interface Observer {
    void update(Observable o, Object arg);
}

```

```java
public class Observable {
    private boolean changed = false; 
    // 让更新不再那么敏感（即我们可以实现隔一段时间后，设置changed为true，才让其接收通知）
    )
    private Vector<Observer> obs; // 使用了线程安全的Vector

    public Observable() {
        obs = new Vector<>();
    }

    public synchronized void addObserver(Observer o) {
        if (o == null)
            throw new NullPointerException();
        if (!obs.contains(o)) {
            obs.addElement(o);
        }
    }
    public synchronized void deleteObserver(Observer o) {
        obs.removeElement(o);
    }

    public void notifyObservers() {
        notifyObservers(null);
    }

    public void notifyObservers(Object arg) {
        /*
         * 一个临时的对象数组，作为当前Observe状态的一个快照
         */
        Object[] arrLocal;

        synchronized (this) {
            /* 这里需要加锁，不加锁可能导致的结果
             * 1、新增加的Observe接收不到通知
             * 2、删除的Observe错误的接收到通知
             */
            if (!changed)
                return;
            arrLocal = obs.toArray();
            clearChanged();
        }

        for (int i = arrLocal.length-1; i>=0; i--)
            ((Observer)arrLocal[i]).update(this, arg);
    }

    public synchronized void deleteObservers() {
        obs.removeAllElements();
    }

    protected synchronized void setChanged() {
        changed = true;
    }

    protected synchronized void clearChanged() {
        changed = false;
    }

    public synchronized boolean hasChanged() {
        return changed;
    }

    public synchronized int countObservers() {
        return obs.size();
    }
}

```

### 简单的Demo

这里以一个报纸公司与其订阅读报者的例子演示观察者模式：

观察者的接口：只需要做自己需要做的事情即可

```java
public interface MyObserve {
    public void updateData();
}

```

主题的接口：一般都有三个方法分别为**注册、移除、通知**

```java
public interface MySubject<E> {
    public void registerObserve(E e);
    public void removeObserve(E e);
    public void notifyObserve();
}

```

报纸出版公司：

```java
public class PaperCom implements MySubject<PaperSub>{
    List<PaperSub> notifyGroup;

    public PaperCom(List<PaperSub> notifyGroup) {
        this.notifyGroup = notifyGroup;
    }

    @Override
    public void registerObserve(PaperSub sub) {
        notifyGroup.add(sub);
    }

    @Override
    public void removeObserve(PaperSub sub) {
        notifyGroup.remove(sub);
    }

    @Override
    public void notifyObserve() {
        for (int i = 0; i < notifyGroup.size(); i++) {
            PaperSub paperSub = notifyGroup.get(i);
            paperSub.updateData();
        }
    }
}

```

报纸订阅者：

```java
public class PaperSub implements MyObserve{
    @Override
    public void updateData() {
        System.out.println("读最新的报纸");
    }
}

```

## 模板模式

## 策略模式

### 定义

> 策略模式：
>
> 定义了**算法族**，分别封装起来，**让它们之间可以互相替换**，此模式**让算法的变化独立于使用算法的客户端**

实际上就是**解耦了策略的定义、创建、使用的三个过程**

### JDK线程池的拒绝策略

JDK线程池有四种拒绝策略，他们都实现了此接口：

```java
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}

```

实现类在`ThreadPoolExecutor`内部，作为静态内部类实现，比如

```java
public static class AbortPolicy implements RejectedExecutionHandler {
    
    public AbortPolicy() { }

    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}

```

### 策略模式的使用

开发中可能会遇到很多`if-else`的逻辑

```java
public class OrderService {
    public double discount(Order order) {
        double discount = 0.0;
        if(order.equals(Order.NORMAL)){
            // ...
        }else if(order.equals(Order.GROUP)){
            // ...
        } else if (order.equals(Order.PROMOTION)) {
            // ...
        }
        return discount;
    }
}

```

使用策略模式+工厂模式我们就可以避免`if-else`嵌套

- 一个策略接口+不同的策略实现
- 一个工厂类，用key代表类型，value存放不同的策略实现

这样就可以避免重复的判断逻辑

## 职责链模式

定义：

> 将请求的发送和接收解耦，让多个接收对象都有机会处理这个请求。
>
> 将这些接收对象串成一条链，并沿着这条链传递这个请求，直到链上的某个接收对象能够处理它为止。

翻译一下的话，就是：多个处理器依次处理同一个请求。一个请求先经过 A 处理器处理，然后再把请求传递给B处理器，B 处理器处理完后再 传递给 C 处理器，以此类推，形成一个链条。

链条上的每个处理器各自承担各自的处理职责，所以叫作职责链模式。 

---

- 比如Spring MVC的处理逻辑，就是一个职责链模式
- 对一些UGC应用（用户生成内容），需要过滤敏感词，这也可以用到职责链模式

## 状态模式

## 迭代器模式

## 访问者模式

## 备忘录模式

## 命令模式

## 解释器模式

## 中介模式