---
title: SpringMVC
date: 2021-06-03 11:49:22
tags:
- Spring
- SpringMVC
categories:
- Spring

---

<center>
引言：

SpringMVC框架学习
</center>

<!--more-->



# SpringMVC

## SpringMVC初识

> SpringMVC是Spring提供的一个实现了Web MVC设计模式的轻量级Web框架

额外需要两个包：

```xml
<!--  SpringMVC框架  -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-web</artifactId>
	<version>4.3.6.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
	<version>4.3.6.RELEASE</version>
</dependency>
```

## SpringMVC工作流程

### 配置MVC四个主要的类

MVC框架内部主要工作由四个构成，无需我们操作，只需要配置即可：

1. `DispatcherServlet`前端控制器
2. `HandlerMapping`处理器映射器
3. `HandlerAdapter`处理器适配器
4. `ViewResolver`视图解析器

以一个简单的demo来看他们的作用：

### 简单的Demo

1. `web.xml`配置文件

   ```xml
   <web-app>
       <servlet>
           <servlet-name>springmvc</servlet-name>
           <servlet-class>
            <!-- 配置前端控制器-->   
               org.springframework.web.servlet.DispatcherServlet
           </servlet-class>
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <!-- 该文件的配置见下面-->
               <param-value>classpath:springmvc-config.xml</param-value>
           </init-param>
           <load-on-startup>1</load-on-startup>
       </servlet>
       <servlet-mapping>
           <servlet-name>springmvc</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   </web-app>
   ```

2. `springmvc-config.xml`配置文件

   ```xml
   <!-- beans中配置 -->
        <!-- handler 处理器：我们在此进行逻辑业务代码的实现 -->
       <bean name="/myController" class="com.dwh.controller.myController" />
        <!-- handlerMapping 处理器映射器配置 -->
       <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
        <!-- handlerAdapter 处理器适配器配置-->
       <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
        <!-- ViewResolver 视图解析器配置 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>
   ```

3. `myController`：继承Controller接口，这就是我们所说的处理器handler

   ```java
   public class myController implements Controller {
       public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
           ModelAndView modelAndView = new ModelAndView();
           modelAndView.addObject("msg","第一个SpringMVC程序");
           modelAndView.setViewName("/WEB-INF/jsp/first.jsp");
           return modelAndView;
       }
   }
   ```

4. `first.jsp`返回的JSP页面

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false"%>
   <html>
   <head>
       <title>Title</title>
   </head>
   <body>
       ${msg}
   </body>
   </html>
   ```

5. tomcat运行，访问`http://localhost:8080/myController`就会返回`第一个SpringMVC程序`这几个字了

### MVC工作流程

如图：（图中橙色标记的是框架内部的实现，白色的（view与handler）是我们程序员需要编写的）

![MVC工作流程图](http://img.yesmylord.cn//image-20210603114926128.png)



流程详述：

1. 用户在浏览器，输入网址`http://localhost:8080/myController`后，请求会被SpringMVC的**前端控制器(DispatcherServlet)拦截**
2. 前端控制器拦截后，调用**处理器映射器（HandlerMapping）**
3. 处理器映射器根据URL，找到具体的处理器，生成**处理器对象及处理器拦截器**，一并返回给前端控制器
4. 前端控制器选择合适的**处理器适配器（HandlerAdapter）**
5. 处理器适配器调用**处理器（Handler）**（也被称为**后端控制器**），返回一个`ModelAndView`对象
6. 返回给HA
7. 返回给DS
8. 根据`ModelAndView`选择合适的**视图解析器（ViewResolver）**
9. VR解析后，向DS发送具体的View
10. DS对View进行**渲染**（将数据填充至视图中）
11. 显示在浏览器，展示给用户

## SpringMVC核心类

### 前端控制器DispatcherServlet配置详解

在`web.xml`中配置

```xml
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-config.xml</param-value>
        </init-param>
        <!-- init-param 配置后，启动时加载对应路径下的配置文件；
		如果不配置此项，默认寻找 servletName-config.xml 文件-->
        <load-on-startup>1</load-on-startup>
        <!-- load-on-startup 为1表示启动时加载；不配置此项表示第一次访问时加载-->
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

### Controller相关注解

#### `@Controller`注解

`@Controller`注解可以帮助我们快速的实现后端控制器，而不需要再实现Controller接口重写handleRequest方法

如下：

```java
@Controller
public class myController{
	....
}
```

#### `@RequestMapping`注解

可以配置在类或是方法上，指明URL访问所调用的处理器

```java
@Controller
@RequestMapping("/hello")
public class myController{
    @RequestMapping("/userService")
    public String userService(Model model){
        model.addAttribute("msg","你好啊");
        return "first";
    }
}
```

这样就可以访问`http://localhost:8080/hello/userService`，就会调用对应方法进行处理

`@RequestMapping`有很多属性，这里提几个重要的属性：

1. name：为映射地址指定别名
2. value：**默认属性**，映射请求到一个方法或类上
3. method：可以选择POST、GET等请求方式

例如：

```java
@RequestMapping("/userService")
```

等同于

```java
@RequestMapping(value = "/userService")
```

而且

```java
@RequestMapping(value = "/userService",method=RequestMethod.GET)
```

等同于

```java
@GetMapping("/userService")
```

PostMapping、PutMapping等同理

### 视图解析器ViewResolver配置

在`springmvc-config.xml`文件中配置：

```xml
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>
```

配置`prefix`与`suffix`就可以更改返回字符串的内容

比如：

```java
@RequestMapping("/userService")
public String userService(Model model){
     model.addAttribute("msg","你好啊");
     return "first";
}
```

SpringMVC支持的返回类型有很多种，最重要的有三种：

1. `ModelAndView`：可以添加Model数据
2. `String`：**跳转页面，但不能携带数据**
3. `Void`：多用在异步请求

String类型只可以跳转页面，配置VR就可以设置它的前缀与后缀，进而访问到对应的Jsp文件

比如这里就访问到：`/WEB-INF/jsp/first.jsp`文件

而且我们也能看到，String虽然不能携带数据，但是我们可以传入Model参数，间接传递数据

### 重定向与请求转发

​		SpringMVC中使用返回类型为String的方法来进行重定向与转发，只需要更改返回值即可

重定向：

```java
public String update(Model model){
    ....
    return "redirect:first";
}
```

请求转发：

```java
public String update(Model model){
    ....
    return "forward:first";
}
```



### 扫描配置

为了使我们的注解起作用，还需要进行配置扫描

如下：

```xml
    <mvc:annotation-driven/>
    <!--  开启Controller等注解  -->
    <context:component-scan base-package="com.dwh" />
```

- 不配置`<mvc:annotation-driven/>`有可能会扫描不到Controller等注解，进而导致404报错
- 注意不要给一个handler映射多个路径

以上可以解决问题，注意`base-package`配置你对应的位置