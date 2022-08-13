---
title: Spring_AspectJ
date: 2020-03-12 22:54:02
tags:
- Spring
categories:
- Spring
---


<center>
引言：

Spring——AspectJ    
</center>

<!-- more -->

# AspectJ

> 一个基于Java语言开发的AOP框架

```xml
<!--SpringAOP相关-->
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

<!--AspectJ-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.9</version>
</dependency>
```
`AspectJ`依赖于原本的AOP，所以需要以上四个

然后再xml文件中配置
```xml
<!-- 开启AspectJ注解开发，自动代理 -->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```
现在我们就能开始愉快的学习AspectJ了


# 注解方式使用AspectJ

AspectJ同样提供了`xml`和注解两种方式，我们先来学习注解方式


## AspectJ提供的不同通知

- `@Before` 前置通知 相当于`BeforeAdvice`
- `@AfterReturning` 后置通知 相当于`AfterReturningAdvice`
- `@Around` 环绕通知 相当于`MethodInterceptor`
- `@AfterThrowing` 异常抛出通知 相当于`ThrowAdvice`
- `@After` 最终通知，不论是否发生异常，该通知都会执行
- `@DeclareParents`引介通知（暂不介绍）


## 通知中定义切点

语法：
```java
execution(<访问修饰符> ? <返回类型> <方法名> (<参数>) <异常>)
//访问修饰符可以省略
```

例如
```java
//1. 匹配所有类public方法
execution(public * *(..))
//表示public 任意返回值* 任意方法名* 任意参数(..) 都会被返回

//2. 匹配指定包下的所有类方法
execution(* com.xxx.xxx.*(..))（不包含子包）
execution(* com.xxx.xxx..*(..))（包含子包，多一个点）
//第一个*表示任意返回类型

//3. 匹配指定类所有方法
execution(* com.xxx.xxx.AnClass.*(..))

//4. 匹配实现特定接口所有类方法
execution(* com.xxx.xxx.AnInterface+.*(..))（这里使用了+号表示所有子类）

//5. 匹配所有save开头的方法
execution(* save*(..))
```

### 前置、后置通知

```xml
<!-- 开启AspectJ注解开发，自动代理 -->
<aop:aspectj-autoproxy />

<bean id="productDao" class="com.dongwenhao.demo11.ProductDao" />

<bean id="myAspectAnno" class="com.dongwenhao.demo11.MyAspectAnno"></bean>
```
切点
```java
public class ProductDao {
    public void save(){
        System.out.println("保存商品");
    }
}
```
切面
```java
@Aspect
public class MyAspectAnno {
    @Before(value = "execution(* com.dongwenhao.demo11.ProductDao.*(..))")
    public void before(){
        System.out.println("前置通知");
    }
    @AfterReturning(value = "execution(* com.dongwenhao.demo11.ProductDao.*(..))")
    public void afterReturning(){
        System.out.println("后置通知");
    }
}
```
测试类
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:AspectJconfig.xml")
public class SpringTest11 {
    @Resource(name = "productDao")
    private ProductDao productDao;

    @Test
    public void test(){
        productDao.save();
    }
}
/*结果
前置通知
保存商品
*/
```
前置通知可以**在方法中传入切入点对象**，后置通知可以传入**方法的返回值对象**
```java
@Aspect
public class MyAspectAnno {
    @Before(value = "execution(* com.dongwenhao.demo11.ProductDao.*(..))")
    public void before(JoinPoint joinPoint){
        System.out.println("前置通知" + joinPoint);
    }
    
    @AfterReturning(value = "execution(* com.dongwenhao.demo11.ProductDao.*(..))", returning = "result")
    public void afterReturning(Object result){
        System.out.println("后置通知" + result);
    }
}
/*结果
前置通知execution(void com.dongwenhao.demo11.ProductDao.save())
保存商品
*/
```

### 环绕通知

```java
@Around(value = "execution(* com.dongwenhao.demo11.ProductDao.*(..))")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("环绕方法执行前");
    Object proceed = joinPoint.proceed();
    System.out.println("环绕方法执行后");
    return proceed;
}
```
环绕通知注解`@Around`，带有一个`ProceedingJoinpoint`，这个对象有一个方法`proceed`，如果不调用`proceed`方法，目标方法将会被拦截

### 异常抛出通知

```java
@AfterThrowing(value = "execution(* com.dongwenhao.demo11.ProductDao.*(..))",throwing = "e")
    public void afterThrowing(Throwable e){
        System.out.println("异常抛出通知" + e.getMessage());
    }

```
正常情况下，异常抛出通知不会执行，只有当异常出现时，这个方法才会调用

可以传入一个`Throwable`对象，获得异常的信息

### 最终通知
无论方法发生了什么，甚至有异常抛出，这个方法总会被执行
```java
@After(value = "execution(* com.dongwenhao.demo11.ProductDao.*(..))")
public void after(){
    System.out.println("最终通知");
}
```


## `@Pointcut`为切点命名
```java
@Pointcut(value = "execution(* com.dongwenhao.demo11.ProductDao.*(..))")
private void myPointcut(){}

@Before(value = "myPointcut()")
public void before(JoinPoint joinPoint){
    System.out.println("前置通知" + joinPoint);
}
```
为了简化开发，切入点不需要一个一个配置，我们可以使用这样的方式，配置一个切入点


格式一般都是这样
```java
@Pointcut(value="execution(...)")
private void pointcut(){}
```


# `XML`方式使用AspectJ

这次我们写一个接口实现类的切点

接口
```java
public interface CustomerDao {
    public void save();
}
```
实现类
```java
public class CustomerImpl implements CustomerDao{
    @Override
    public void save() {
        System.out.println("保存数据");
    }
}
```
切面
```java
public class MyAspectXml {
    public void before(){
        System.out.println("前置通知");
    }
}
```
配置
```
<bean id="customerDao" class="com.dongwenhao.demo12.CustomerImpl" />

<bean id="aspectXml" class="com.dongwenhao.demo12.MyAspectXml"></bean>

<aop:config>
    <aop:pointcut id="pointcut1" expression="execution(* com.dongwenhao.demo12.CustomerDao.*(..))" />
    <aop:aspect ref="aspectXml">
        <aop:before method="before" pointcut-ref="pointcut1"></aop:before>
    </aop:aspect>
</aop:config>
```
这里以前置通知示例，其他通知同理可以写出


