---
title: Spring_Bean
date: 2020-03-08 23:22:00
tags:
- Spring
categories:
- Spring
---


<center>
引言：
Spring_配置Bean

</center>

<!--more-->

# Spring工厂

`Spring`工厂全家福一览

![image](https://github.com/YesYourHighness/MyPicStore/raw/master/Spring/springFactory.png)


看下面的代码
```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
UserService usrService = (UserService) applicationContext.getBean("usrService");
usrService.sayHello();
```
我们用到了两个工厂`ApplicationContext`、`ClassPathXmlApplicationContext`

在老版本的`Spring`中，用的工厂其实是`BeanFactory`，`ApplicationContext`是后来版本的工厂类，`ApplicationContext`除了具有`BeanFactory`的所有方法外，他还具有
- Easier integration with Spring’s AOP features
    更容易与 Spring 的 AOP 特性集成

- Message resource handling (for use in internationalization)
消息资源处理(用于国际化)

- Event publication
活动刊物

- Application-layer specific contexts such as the WebApplicationContext for use in web applications.
应用层特定的上下文，例如 web 应用程序中使用的 WebApplicationContext。

他们创建Bean的实例的时机也是不一样的
- `BeanFactory`会在调用`getBean()`方法时，才会创建这个Bean的实例
- `ApplicationContext`在读取Spring配置文件时，就会把配置文件中**所有单例模式**生成的类全部生成

工厂对比：
```java
@Test
public void springTest1(){
    /**
     * 使用Spring
     */
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserService usrService = (UserService) applicationContext.getBean("usrService");
    usrService.sayHello();
}
@Test
public void springTest2(){
    BeanFactory bf = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
    //XmlBeanFactory()这个方法已经淘汰
    UserService usrService = (UserService) bf.getBean("usrService");
    usrService.sayHello();
}
```


# Bean管理

`Spring`管理`Bean`的方式主要有两种，分别是XML和注解

我们分别来看这两种

# XML

`Spring`实例化`Bean`的三种方法

- 使用类构造器实例化（默认无参数）
- 使用静态工厂方法实例化（简单工厂方法）
- 使用实例工厂方法实例化（工厂方法模式）

### 使用类构造器实例化
```java
public class Bean1 {
    public Bean1() {
        System.out.println("Bean1被实例化了");
    }
}
```
配置文件
```xml
<bean id="bean1" class="com.dongwenhao.demo02.Bean1" />
```
测试
```java
@Test
public void test1(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
    Bean1 bean1 = (Bean1) applicationContext.getBean("bean1");
}
```
结果
```
log4j:WARN No appenders could be found for logger (org.springframework.core.env.StandardEnvironment).
log4j:WARN Please initialize the log4j system properly.
Bean1被实例化了
```

### 使用静态工厂方法实例化
```java
public class Bean2 {
    Bean2(){
        System.out.println("Bean2被创建了");
    }
}
```
工厂
```java
public class Bean2Factory {
    public static Bean2 getBean2(){
        return new Bean2();
    }
}
```

配置文件
```xml
<bean id="bean2" class="com.dongwenhao.demo02.Bean2Factory" factory-method="getBean2" />
```
测试
```java
@Test
public void test2(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
    Bean2 bean2 = (Bean2) applicationContext.getBean("bean2");
}
```
结果
```
log4j:WARN No appenders could be found for logger (org.springframework.core.env.StandardEnvironment).
log4j:WARN Please initialize the log4j system properly.
Bean2被创建了
```


### 使用实例工厂方法实例化
```java
public class Bean3 {
    Bean3(){
        System.out.println("Bean3被创建了");
    }
}
```
工厂类
```java
public class Bean3Factory {
    public Bean3 getBean3(){
        return new Bean3();
    }
}
```

配置文件
```xml
<bean id="bean3Factory" class="com.dongwenhao.demo02.Bean3Factory" />
<bean id="bean3" factory-bean="bean3Factory" factory-method="getBean3" />
```
测试
```java
@Test
public void test3(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
    Bean3 bean3 = (Bean3) applicationContext.getBean("bean3");
}
```
结果
```
log4j:WARN No appenders could be found for logger (org.springframework.core.env.StandardEnvironment).
log4j:WARN Please initialize the log4j system properly.
Bean3被创建了
```

## Bean的配置
```xml
<bean />
```
## id 和 name
- 一般情况下我们使用`id`而不是`name`
- `id`属性在`IOC`必须是唯一的
- 如果`Bean`名称有特殊字符，就需要使用`name`属性


## class

用来设置一个类的完全路径名称，主要作用是IOC容器生成类的实例

## scope
定义`Bean`的作用域，值有四个

- `singleton`：在SpringIOC容器中仅存在一个`Bean`实例，`Bean`以单例的方式存在
- `prototype`：每次调用`getBean()`时都会返回一个新的实例
- `request`：每次HTTP请求都会创建一个新的`Bean`（仅适用在WebApplicationContext环境）
- `session`：同一个`HTTP Session`共享一个`Bean`，不同的`HTTP Session`使用不同的`Bean`（仅适用在WebApplicationContext环境）


# Bean的生命周期
![image](https://github.com/YesYourHighness/MyPicStore/raw/master/Spring/lifeCycle.png)


为了体验一个完整的生命周期，我们需要建两个类
```java
public class Man implements BeanNameAware, ApplicationContextAware, InitializingBean, DisposableBean {
    private String name;

    public Man() {
        System.out.println("第一步：实例化");
    }

    public void setName(String name) {
        System.out.println("第二步：设置属性");
        this.name = name;
    }
    @Override
    //实现BeanNameAware接口
    public void setBeanName(String name){
        System.out.println("第三步：设置bean的name："+name);
    }

    @Override
    //实现ApplicationContextAware接口
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("第四步：了解工厂信息");
    }
    
    // 第五步和第八步在另一个类中 
    
    @Override
    //实现InitializingBean接口
    public void afterPropertiesSet() throws Exception {
        System.out.println("第六步：属性设置后");
    }
    
    //初始化方法，需要在xml中指出
    public void setup(){
        System.out.println("第七步：Man被初始化了");
    }
    //类自身的方法
    public void run(){
        System.out.println("第九步：执行自身的业务方法");
    }
    @Override
    //实现DisposableBean接口
    public void destroy() throws Exception {
        System.out.println("第十步：执行Spring的销毁方法");
    }
    //销毁方法，需要在xml中指出
    public void teardown(){
        System.out.println("第十一步：Man被销毁了");
    }
}
```
```java
public class MyBeanPostProcessor implements BeanPostProcessor {
    /*
    实现BeanPostProcessor接口
    第五步和第八步是最重要的两步！！
    之后我们详细说
    */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("第五步：初始化前方法");
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("第八步：初始化后方法");
        return bean;
    }
}
```
配置文件
```xml
<bean id="man"
          class="com.dongwenhao.demo03.Man"
          init-method="setup"
          destroy-method="teardown"
    >
    <property name="name" value="小王" />
</bean>
<bean class="com.dongwenhao.demo03.MyBeanPostProcessor" />
<!--不需要配置id，这个类会在Spring生成类的实例时自动调用-->
```
测试方法
```java
@Test
public void test() {
    ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
    Man man = (Man) ac.getBean("man");
    man.run();
    ac.close();
}
```


打印结果
```
log4j:WARN No appenders could be found for logger (org.springframework.core.env.StandardEnvironment).
log4j:WARN Please initialize the log4j system properly.
第一步：实例化
第二步：设置属性
第三步：设置bean的name：man
第四步：了解工厂信息
第五步：初始化前方法
第六步：属性设置后
第七步：Man被初始化了
第八步：初始化后方法
第九步：执行自身的业务方法
第十步：执行Spring的销毁方法
第十一步：Man被销毁了
```

生命周期中，重要的就是第五步和第八步，这个阶段可以增强我们的方法

# BeanPostProcessor

可以在生成类的过程中对类产生代理，并且可以对其中的方法进行增强

例如这么一个类
```java
public interface UserDao {
    void findAll();
    void save();
    void update();
    void delete();
}

public class UserDaoImpl implements UserDao{
    @Override
    public void findAll() {
        System.out.println("查询用户");
    }

    @Override
    public void save() {
        System.out.println("保存用户");
    }

    @Override
    public void update() {
        System.out.println("修改用户");
    }

    @Override
    public void delete() {
        System.out.println("删除用户");
    }
}
```
怎么增强一个类的方法呢？

这里我们比如说给`save()`方法运行前，还需要进行校验

这里就需要用到我们的`BeanPostProcessor`接口了


```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("第五步：初始化前方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(final Object bean, String beanName) throws BeansException {
        System.out.println("第八步：初始化后方法");
        if("userDao".equals(beanName)){
            Object proxy = Proxy.newProxyInstance(bean.getClass().getClassLoader(), bean.getClass().getInterfaces(), new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    if("save".equals(method.getName())){
                        System.out.println("权限校验");
                        //权限校验使用打印来代替一下
                        return method.invoke(bean,args);
                    }
                    return method.invoke(bean,args);
                }
            });
            return proxy;
        }
        return bean;
    }
}
```
xml配置
```xml
<bean class="com.dongwenhao.demo03.MyBeanPostProcessor" />
<!--不需要配置id，这个类会在Spring生成类的实例时自动调用-->
<bean id="userDao" class="com.dongwenhao.demo03.UserDaoImpl" />
```

测试类：
```java
@Test
public void test2() {
    ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserDao userDao = (UserDao) ac.getBean("userDao");
    userDao.findAll();
    userDao.delete();
    userDao.update();
    userDao.save();
}
```
打印结果：
```java
log4j:WARN No appenders could be found for logger (org.springframework.core.env.StandardEnvironment).
log4j:WARN Please initialize the log4j system properly.
第五步：初始化前方法
第八步：初始化后方法
查询用户
删除用户
修改用户
权限校验
保存用户
```

# 属性注入

对于类的成员变量，我们一般有三种办法来存储他们的实例
- 构造函数
- `setter`方法
- 接口注入

在Spring中，支持上面两种方法

## 构造方法注入


假设有一个类
```java
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
在构造方法中传入参数，我们只需要增加一个`constructor-arg`标签即可

成员变量名用`name`，值使用`value`
```java
<bean id="user" class="com.dongwenhao.demo04.User">
    <constructor-arg name="name" value="匿名" />
    <constructor-arg name="age" value="0" />
</bean>
```

## `setter`方法注入
```java
public class User {
    private String name;
    private int age;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
我们只需要增加一个`property`标签即可

成员变量名用`name`，值使用`value`
```xml
<bean id="user" class="com.dongwenhao.demo04.User">
    <property name="name" value="匿名"></property>
    <property name="age" value="0"></property>
</bean>
```

如果有一个成员变量时类怎么办？

```java
public class Cat {
    private String name;

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
public class User {
    private String name;
    private int age;
    private Cat cat;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setCat(Cat cat) {
        this.cat = cat;
    }
}
```
因为`Cat`也是一个类，我们也需要配置，而且实际上在`User`中，`Cat`其实还是`set`方法来注入的，所以我们仍然使用`property`

但是有一个区别，对于普通的值我们可以使用`value`给值，但是对于配置了的类，我们需要使用`ref`

像这样：
```xml
<bean id="user" class="com.dongwenhao.demo04.User">
    <property name="name" value="匿名"></property>
    <property name="age" value="0"></property>
    <property name="cat" ref="cat" ></property>
</bean>
<bean id="cat" class="com.dongwenhao.demo04.Cat">
    <property name="name" value="小橘猫"></property>
</bean>
```
使用构造方法注入时，也是使用`ref`


## 引入p命名
其实上述的`xml`配置都可以通过`p`来简化，例如可以这样
```xml
<bean id="user" class="com.dongwenhao.demo04.User" p:name="小明" p:age="18" p:cat-ref="cat"></bean>
<bean id="cat" class="com.dongwenhao.demo04.Cat" p:name="Kitty"></bean>
```
但是直接改会报错的，我们必须在`beans`中增加这么一个
```
xmlns:p="http://www.springframework.org/schema/p"
```

## SpEl注入
> SpEL：spring expression language  spring表达式语言

语法：`#{表达式}`

例如：其中`category`和` productInfo`都是类
```xml
<bean id ="category" class="com.dongwenhao.demo04.Category">
    <property name="name" value="#{'男装'}"></property>
</bean>

<bean id = "productInfo" class="com.dongwenhao.demo04.ProductInfo"></bean>

<bean id="product" class="com.dongwenhao.demo04.Product">
    <property name="name" value="#{'外套'}"></property>
    <!--注意字符串要加引号-->
    <property name="price" value="#{ productInfo.backPrice()}"></property>
    <!--这样可以通过成员方法来给值-->
    <property name="category" value="#{category}"></property>
    <!--成员变量是类直接写就可以-->
</bean>
```

## 复杂类型注入
假如一个类像这样，该怎么注入呢？
```java
public class CollectionBean {
    private String[] arr;
    private List<String> list;
    private Set<String> set;
    private Map<String,Integer> map;
    private Properties;
    ...(省略set方法)
}
```
注入类型
```xml
<bean id="collectionBean" class="com.dongwenhao.demo04.CollectionBean">
    <!--数组注入-->
    <property name="arr">
        <list>
            <value>a</value>
            <value>b</value>
            <value>c</value>
        </list>
    </property>
    <!--list注入，和数组注入一样-->
    <property name="list">
        <list>
            <value>a</value>
            <value>b</value>
            <value>c</value>
            <!--如果这里是引用类型，要使用ref-->
        </list>
    </property>
    <property name="set">
        <set>
            <value>a</value>
            <value>b</value>
            <value>c</value>
        </set>
    </property>
    <property name="map">
        <map>
            <entry key="a" value="1"/>
            <entry key="b" value="2"/>
            <entry key="c" value="3"/>
            <!--map集合内使用 entry-->
        </map>
    </property>
    <property name="properties">
        <props>
            <prop key="username">root</prop>
            <prop key="password">123</prop>
        </props>
    </property>
</bean>
```

# 注解方式
需要引入约束

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 开启注解扫描   -->
     <context:component-scan base-package="com.dongwenhao.demo05"/>
</bean>
```
在加载此`xml`文件时，开启注解扫描，就会去对应包下找类

```java
@Component("userService")
public class UserService {
    public void sayHello(String name){
        System.out.println("你好："+name);
    }
}
```
这句话`@Component("userService")`相当于配置xml文件`<bean id="userService" class="xxx/>`


为了清晰的标注类本身的用途，Spring提供了3个功能基本和`@Component`等效的注解
- `@Respository`用于对DAO实现类进行标注
- `@Service`用于对Service实现类进行标注
- `@Controller`用于对Controller实现类进行标注


### 复杂的注解注入

```java
@Repository("userDao")
public class UserDao {
    public void save(){
        System.out.println("数据保存");
    }
}


@Repository("userService")
public class UserService {
    /**
     * @Value
     * 如果有set方法就可以加在set方法上，没有set方法就加在属性上
     */
    @Value("肉包子")
    private String food;
    /**
     * @Autowired
     * 自动注入，默认按照类型匹配，忽略名称；如果类型相同，则按名称注入
     * 可以加在set方法上，也可以加在变量上
     * @Qualifier("name")
     * 保证名称也得相同，假如UserDao的注释是@Repository("userDao123456")
     * 而这里是userDao，那么便不会匹配
     * @Resource
     * 有一个name属性，使用@Resource(name="userDao")，相当于同时使用了上面两个
     */
    //@Autowired
    //@Qualifier("userDao")
    @Resource(name="userDao")
    private UserDao userDao;

    public void sayHello(String name){
        System.out.println("你好："+name);
    }
    public void eat(){
        System.out.println("eat" + food);
    }

    public void save(){
        System.out.println("Service执行保存");
        userDao.save();
    }
}
```

### 生命周期的注解
用到生命周期时，使用`xml`方式是这样的
```xml
<bean id="man"
          class="com.dongwenhao.demo03.Man"
          init-method="setup"
          destroy-method="teardown"
    >
    ...
</bean>
```

使用注解是什么样的呢？
```java
/**
 * @Scope("xx")
 * 设置这个类的模式：
 * - singleton 单例模式
 * - prototype 多例模式
 * - request 每次请求都会返回新的对象（仅适用在WebApplicationContext环境）
 * - session 一个会话session内，返回同一个对象（仅适用在WebApplicationContext环境）
 */
@Component("bean")
@Scope("prototype")
public class Bean {
    @PostConstruct
    public void init(){
        System.out.println("初始化");
    }
    public void run(){
        System.out.println("运行");
    }

    /**
     * @PreDestroy
     * 这个注解只当单例模式singleton有效
     */
    @PreDestroy
    public void destroy(){
        System.out.println("销毁");
    }
}
```

## 传统`xml`和注解配合使用
我们回顾`xml`和注解，发现他们其实各有优劣

- xml方式： 结构清晰，易于阅读
- 注解方式：开发便捷，注入方便

如果我们`bean`注入使用`xml`，而属性注入使用注解，是不是很方便呢

混合使用他们，我们得先
1. 引入context命名空间
2. 在配置文件中添加`context:annotation-config`标签

```xml
<!--<context:component-scan base-package="xxx"/>-->
<!--我们之前使用的是包扫描，它会扫描包下的注解，下方的这个标签其实是包扫描的一部分-->
<context:annotation-config />
<!--这个注解只能识别那些属性注入的注解-->

<bean id="productService" class="com.dongwenhao.demo05.ProductService" />
<bean id="category" class="com.dongwenhao.demo05.CategoryDao" />
<bean id="userDao" class="com.dongwenhao.demo05.UserDao" />
```

```java
public class ProductService {
    @Resource(name = "userDao")
    private UserDao userDao;
    @Resource(name = "category")
    private CategoryDao categoryDao;
    public void save(){
        System.out.println("Product开始保存");
        categoryDao.save();
        userDao.save();
    }
}
```

