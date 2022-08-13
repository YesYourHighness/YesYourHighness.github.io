---
title: Spring_AOP
date: 2020-03-10 21:25:05
tags:
- Spring
categories:
- Spring
---

<center>
引言：

Spring——AOP
</center>
<!-- more -->

# AOP
> AOP：Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术

在传统的开发中，如果我们想对一个方法进行增强（加强这个方法的功能：例如给保存功能加权限校验）
，就需要给这个方法中再加一个方法，但是如果这样的类非常多，有成百上千个，我们该怎么办？

传统方式使用**纵向继承**，即子类继承父类，父类中有权限校验的代码，于是子类也可以调用

AOP采用**横向抽取机制**，即通过动态代理Proxy来实现对代码的增强

而`Spring AOP` 就是使用纯JAVA语言实现的，不需要专门的编译过程和类加载器，在运行期通过代理方式向目标类植入增强代码

# Spring AOP

相关术语
> Joinpoint：（连接点）指被拦截到的点，spring中这些点是方法，因为Spring只支持方法类型的连接点

> Pointcut：（切入点）指我们要对哪些Joinpoint进行拦截

> Advice：（通知/增强）指拦截到Joinpoint类之后要做的事情，分为前置通知、后置通知、异常通知、最终通知、环绕通知（切面要完成的功能）

> Introduction：（引介）一种特殊的通知，在不修改类代码的前提下、Introduction可以在运行期间，动态的添加一些方法或Field

> Target：（目标对象）增强的目标，代理的目标对象

> Weaving：（织入）把Advice应用到Target来创建新的代理对象的过程，Spring采用动态代理来织入，而AspectJ采用编译期织入和类装载期织入

> Proxy（代理）：一个类被AOP织入增强后，就产生一个结果代理类

> Aspect（切面）：是切入点和通知（引介）的结合


# 代理

## 使用JDK动态代理
我们现在使用动态代理来增强这个类的`save`方法

```java
public interface UserDao {
    void save();
}
```
实现类
```java
public class UserDaoImpl implements UserDao {
    @Override
    public void save() {
        System.out.println("保存数据");
    }
}
```
代理类
```java
public class MyProxy implements InvocationHandler {

    private UserDao userDao;

    public MyProxy(UserDao userDao) {
        this.userDao = userDao;
    }

    public Proxy creatProxy() {
        /**
         * proxy的newInstance方法需要传入三个参数
         * 第一个：类加载器
         * 第二个：接口
         * 第三个：InvocationHandler的实现类，这个类中有invoke方法，我们需要用这个方法来增强我们的方法
         *        在这里传入了this，是因为这个类就实现了InvocationHandler接口
         */
        Proxy proxy = (Proxy) Proxy.newProxyInstance(
                userDao.getClass().getClassLoader(),
                userDao.getClass().getInterfaces(),
                this);
        return proxy;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //如果调用的是save方法，就进行权限校验
        if("save".equals(method.getName())){
            System.out.println("权限校验");
            return method.invoke(userDao,args);
        }
        return method.invoke(userDao,args);
    }
}
```
测试类
```java
@Test 
public void test1(){
    UserDao userDao = new UserDaoImpl();
    UserDao userDaoProxy = (UserDao) new MyProxy(userDao).creatProxy();
    userDaoProxy.save();
}
//结果
/*
权限校验
保存数据
*/
```

**我们发现JDK的动态代理，只能适用于实现了接口的类**，对于没有实现接口的类，我们没法使用JDK的动态代理


## 使用CGLIB生成代理
原理是：针对目标产生了一个子类

```java
public class ProductDao {
    public void save() {
        System.out.println("数据保存");
    }
}
```

```java
/**
 * 注意是org.springframework.cglib.proxy这个包下的MethodInterceptor
 */
public class MyCglibProxy implements MethodInterceptor {
    private ProductDao productDao;

    public MyCglibProxy(ProductDao productDao) {
        this.productDao = productDao;
    }

    public Object creatProxy(){
        //1. 核心类
        Enhancer enhancer = new Enhancer();
        //2. 设置父类
        enhancer.setSuperclass(productDao.getClass());
        //3. 设置回调，类似于InvocationHandler这个类
        enhancer.setCallback(this);
        Object proxy = enhancer.create();
        return proxy;
    }

    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        if("save".equals(method.getName())){
            System.out.println("权限检验");
            return methodProxy.invokeSuper(proxy,args);
        }
        return methodProxy.invokeSuper(proxy,args);
    }
}
```

## 代理总结

1. `Spring`在运行期生成动态代理对象，不需要特殊的编译器
2. `Spring AOP`的底层就是通过JDK动态代理或CGLib动态代理技术为目标进行横向织入
3. 若目标实现了接口，则使用JDK的`Proxy`
4. 目标没有实现接口，则使用Spring的`CGLib`
5. 编程中尽量优先使用接口创建动态代理、便于解耦
6. 对于使用`final`的方法或类无法代理  
7. `Spring`只支持方法连接点、不提供属性连接点


# SpringAOP 增强（通知）类型

> Advice：（通知/增强）指拦截到Joinpoint类之后要做的事情，分为前置通知、后置通知、异常通知、最终通知、环绕通知（切面要完成的功能）

AOP联盟定义了五个Advice类型，主要有：
1. 前置通知`org.springframework.aop.MethodBeforeAdvice`，在目标方法执行前实施增强
2. 后置通知`org.springframework.aop.AfterReturningAdvice`，在目标方法执行后实施增强
3. 环绕通知`org.aopalliance.intercept.MethodInterceptor`，在目标方法执行前、执行后实施增强
4. 异常抛出通知`org.springframework.aop.IntroductionInterceptor`，在方法抛出异常后实施增强
5. 引介通知，`org.springframework.aop.IntroductionInterceptor`，在目标类中添加一些新的方法和属性

**注意：**Spring只有对于方法的连接点，而没有对于属性的连接点，所以Spring没有引介通知


# SpringAOP切面类型
> Joinpoint：（连接点）指被拦截到的点，spring中这些点是方法，因为Spring只支持方法类型的连接点

> Pointcut：（切入点）指我们要对哪些Joinpoint进行拦截

> Aspect（切面）：是切入点和通知（引介）的结合




## 切面配置
为了使用切面，除了引入`beans core context expression`之外，还需要引入
```xml
<dependency>
    <groupId>aopalliance</groupId>
    <artifactId>aopalliance</artifactId>
    <version>1.0</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
```

实际上我们并不需要自己去写代理类，Spring为我们提供了代理类`ProxyFactoryBean`，我们只需要配置它的信息即可
```
// 可配置属性有：
- target 代理的对象
- proxyInterfaces 代理要实现的接口，接口很多时使用List来赋值
- interceptorNames 需要织入目标的Advice（拦截名）
- proxyTargetClass 是否对类代理而不是接口，设置为true时，使用CGLib代理
- optimize 设置为true时，强制使用CGLib
``` 
## 使用Advisor一般切面

1. `Advisor`：一般的切面，Advice（通知）本身就是一个切面，它会对对目标类**所有方法**进行拦截

2. `PointcutAdvicor`：具有切入点的切面，可以指定拦截

3. `IntroductionAdvisor`：代表引介切面、针对引介通知而使用切面

同样，由于Spring原因，所以没有`IntroductionAdvisor`

以下是一个小例子

----


配置
```xml
<bean id="studentDao" class="com.dongwenhao.demo08.StudentDaoImpl" />

<bean id="myBeforeAdvice" class="com.dongwenhao.demo08.MyBeforeAdvice" />

<bean id="studentProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <!--配置目标类-->
    <property name="target" ref="studentDao"/>
    <!--配置接口-->
    <property name="proxyInterfaces" value="com.dongwenhao.demo08.StudentDao"/>
    <!--配置拦截的名称-->
    <property name="interceptorNames" value="myBeforeAdvice" />
</bean>
```
接口
```java
public interface StudentDao{
    public void save();
}   
```
类
```java
public class StudentDaoImpl implements StudentDao {
    @Override
    public void save() {
        System.out.println("保存信息");
    }
}
```
使用前置通知
```java
public class MyBeforeAdvice implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
            System.out.println("权限校验");
    }
}
```

## 使用 PointcutAdvisor 切点切面

带有切点的切面可以对特定的切面进行处理

常用`PointcutAdvisor`实现类
1. `DefaultPointcutAdvisor` 最常用的切面类型，可以通过任意`Pointcut`和`Advice`组合定义切面
2. `JdkRegexpMethodPointcut`构造正则表达式切点

以下是例子

----

这个例子没有接口，使用CGLib，且使用环绕通知

配置
```xml
<bean id="customerDao" class="com.dongwenhao.demo09.CustomerDao" />
<bean id="myAroundAdvice" class="com.dongwenhao.demo09.MyAroundAdvice" />
<!--使用正则表达式切点-->
<bean id="myAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="pattern" value=".*" />
    <!--这里配置正则表达式-->
    <property name="advice" ref="myAroundAdvice" />
    <!--这里配置引用-->
</bean>

<bean id="customerDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="customerDao" />
    <property name="proxyTargetClass" value="true" />
    <!--设置为true，即使用CGLib-->
    <property name="interceptorNames" value="myAdvisor" />
</bean>
```
环绕通知
```java
/**
 * 这里的接口是org.aopalliance.intercept.MethodInterceptor;
 */
public class MyAroundAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("环绕前->->->");
        Object proceed = invocation.proceed();
        System.out.println("<-<-<-环绕后");
        return proceed;
    }
}
```

```java
public class CustomerDao {
    public void save() {
        System.out.println("保存数据");
    }
}
```

# 自动创建代理

每个代理我们都使用`ProxyFactoryBean`织入切面代理的话，Bean的数量一多，工作量会非常的大

自动创建代理的类有以下：
- `BeanNameAutoProxyCreator` 根据Bean名称创建代理
- `DefaultAdvisorAutoProxyCreator` 根据Advisor本身包含信息创建代理
- `AnnotationAwareAspectJAutoProxyCreator` 基于Bean中的AspectJ注解进行自动代理

只需要简单的配置，我们就能完成这项工作了！


## BeanNameAutoProxyCreator
```xml
<!--只需要配置
    1. 目标
    2. 增强（Advice）   
-->
<bean id="customerDao" class="com.dongwenhao.demo10.CustomerDao" />
<bean id="studentDao" class="com.dongwenhao.demo10.StudentDaoImpl"/>
<!--目标类-->

<bean id="myAroundAdvice" class="com.dongwenhao.demo10.MyAroundAdvice" />
<bean id="myBeforeAdvice" class="com.dongwenhao.demo10.MyBeforeAdvice"/>
<!--通知-->

<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames" value="*Dao" />
    <!--为所有以Dao结尾的Bean自动代理-->
    <property name="interceptorNames" value="myAroundAdvice" />
    <!--选择通知-->
</bean>
<!--自动代理-->
```

## DefaultAdvisorAutoProxyCreator

```xml
<bean id="customerDao" class="com.dongwenhao.demo10.CustomerDao" />
<bean id="studentDao" class="com.dongwenhao.demo10.StudentDaoImpl"/>

<bean id="myAroundAdvice" class="com.dongwenhao.demo10.MyAroundAdvice" />
<bean id="myBeforeAdvice" class="com.dongwenhao.demo10.MyBeforeAdvice"/>

<bean id="myAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="pattern" value="com.dongwenhao.demo10.StudentDaoImpl.save" />
    <!--这里要配置到方法-->
    <property name="advice" ref="myAroundAdvice" />
    <!--选择通知-->
</bean>

<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>
```
 




