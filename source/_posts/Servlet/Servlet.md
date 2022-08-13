---
title: Servlet
date: 2019-10-11 22:09:11
tags: 
- Servlet
categories: 
- 后台
- Servlet
---

<center>

引言：

Servelet

</center>


<!--more-->

-----

# Servlet

server applet

## 概念：

- 运行在服务端的小程序
- 一个接口，定义了java类被浏览器访问到(tomcat识别)的规则
- 将来我们自定义一个类，实现Servlet接口，复写方法


## 快速入门
1. 创建一个**JavaEE的项目**
2. 定义一个类，实现Servelet接口
3. 实现接口中的抽象方法（共五个）
    ```java
    /**
     * @author BlackKnight
     */
    public class servletDemo implements Servlet {
        @Override
        public void init(ServletConfig servletConfig) throws ServletException {
    
        }
    
        @Override
        public ServletConfig getServletConfig() {
            return null;
        }
    
    
        @Override
        //提供服务的方法
        public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
            System.out.println("Hello Servlet");
        }
    
        @Override
        public String getServletInfo() {
            return null;
        }
    
        @Override
        public void destroy() {
    
        }
    }

    ```
4. 配置Servelet（配置`web.xml`文件）
    ```xml
   <!--配置Servlet-->
    <servlet>
        <servlet-name>demo1</servlet-name>
        <servlet-class>servletDemo</servlet-class>
    <!--配置全类名，src目录下一层一层写-->
    </servlet>
    <!--配置网页虚拟路径-->
    <servlet-mapping>
        <servlet-name>demo1</servlet-name>
        <url-pattern>/demo1</url-pattern>
    </servlet-mapping>
    ```
5. 启动服务器，打开`http://localhost:8080/demo1`


## Servlet 执行原理

servlet执行的步骤：

浏览器输入`/demo1`  -->  Servlet自动匹配xml文件，找到xml文件中`<url-pattern>`的`/demo1` ，匹配`<servlet-name>`

-->  匹配到`<servlet-name>`后，又由此匹配到`<servlet-class>`  --> Tomcat通过这里的全类名对应的字节码文件加载进入内存

--> 由此通过反射机制创建对象 --> 调用`Service`方法

---
对象的创建和调用方法都依赖于web容器Tomcat来执行，不需要我们操作


## Servlet方法
```java
public class servletDemo implements Servlet {
    /**
     * 初始化方法
     * 在Servlet被创建后只会执行一次
     * @param servletConfig
     * @throws ServletException
     */
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    /**
     * 获取ServletConfig对象
     * ServletConfig，Servlet的配置对象
     * @return
     */
    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    /**
     * 提供服务的方法
     * 每一次Servlet被访问时，就会调用一次此方法，执行多次
     * @param servletRequest
     * @param servletResponse
     * @throws ServletException
     * @throws IOException
     */
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("Hello Servlet");
    }

    /**
     * 获取Servlet的一些信息的方法：版本，作者...
     * 一般不会实现
     * @return
     */
    @Override
    public String getServletInfo() {
        return null;
    }

    /**
     * 销毁方法
     * 在Servlet正常关闭时调用，执行一次
     */
    @Override
    public void destroy() {

    }
}

```

### 生命周期
1. 被创建：执行init方法，只执行一次
    
    - 默认情况下第一次被访问时Servlet被创建
    - 可以配置执行Servlet的创建时机
    ```xml
    <!--在Servlet标签下配置此内容-->
    <!--配置全类名,src目录下一层一层写-->
    <!--可以指定Servlet的创建时机
        1. 第一次被访问时创建（值为负数，默认即为-1）
        2. 在服务器启动时创建（值为0或者正整数）
    -->
    <load-on-startup>-1</load-on-startup>
    ```
    - Servlet的init方法，只执行一次，说明一个Servlet在内存中只存在一个对象，Servlet是单例的：这存在一个安全问题，多个对象同时访问了共享资源，可能出现线程安全问题
    - 解决：尽量不要在Servlet中定义成员变量，可以在方法内定义变量，即使定义了成员变量，也不要对其修改
    
2. 提供服务：执行servlet方法，执行多次
    - 每次访问Servlet时，Servlet方法都会被调用一次

3. 被销毁：执行destroy方法，正常关闭时调用，也只执行一次
    - Servlet被销毁时（销毁前）执行。服务器关闭时，Servlet会被销毁
    - 只有正常关闭时才会执行destroy方法

# Servlet3.0

好处：无需再配置web.xml

步骤：
1. 创建JavaEE项目，选择Servlet版本3.0以上，可以不创建webxml
2. 定义一个类实现Servlet接口
3. 复写方法
4. 在类上使用@WebServelt注解进行配置
    ```java
    @WebServlet(urlPatterns = "/demo")
    //可以这样配置url
    @WebServlet("/demo")
    //由于urlPatterns是value值，所以可以省略，直接写url即可
    ```


# IDEA与Tomcat的相关配置

- IDEA会为每一个tomcat部署的项目单独建立一个配置文件，

查看控制台此配置信息，就是配置文件所在地
```
Using CATALINA_BASE:
```

- 工作空间项目 和 tomcat部署的web项目文件

    - tomcat实际访问的是tomcat“tomcat部署的web项目”，它位于工作空间项目下的web目录下的所有资源
    - WEB-INF 目录下的资源不能被浏览器直接访问


# Servlet体系结构

Servlet接口下有一个抽象实现类`GenericServlet 抽象类`

此类下还有一个子类
`HttpServlet 抽象类`


## GenericServlet
只有`Service()`方法进行了抽象，其他都是空方法

所以我们继承此接口只需要重写一个方法

其他的方法想要重写也是可以的
```java
public class servletDemo extends GenericServlet {
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {

    }
}
```

## HttpServlet

我们常在开发中使用此类，对Http协议的一种封装可以简化操作

此类封装了HTTP并给出了Service等方法
```java
//Service()方法
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String method = req.getMethod();
    //1. 接收方法的类型
    long lastModified;
    //2. 判断该方法是什么方法,共有七个方法，用字符串类的equals来判断等价
    if (method.equals("GET")) {
        lastModified = this.getLastModified(req);
        if (lastModified == -1L) {
            this.doGet(req, resp);
            //3. 调用响应的doGet或者doPost方法
        } else {
            long ifModifiedSince;
            try {
                ifModifiedSince = req.getDateHeader("If-Modified-Since");
            } catch (IllegalArgumentException var9) {
                ifModifiedSince = -1L;
            }

            if (ifModifiedSince < lastModified / 1000L * 1000L) {
                this.maybeSetLastModified(resp, lastModified);
                this.doGet(req, resp);
            } else {
                resp.setStatus(304);
            }
        }
    } else if (method.equals("HEAD")) {
        lastModified = this.getLastModified(req);
        this.maybeSetLastModified(resp, lastModified);
        this.doHead(req, resp);
    } else if (method.equals("POST")) {
        this.doPost(req, resp);
    } else if (method.equals("PUT")) {
        this.doPut(req, resp);
    } else if (method.equals("DELETE")) {
        this.doDelete(req, resp);
    } else if (method.equals("OPTIONS")) {
        this.doOptions(req, resp);
    } else if (method.equals("TRACE")) {
        this.doTrace(req, resp);
    } else {
        String errMsg = lStrings.getString("http.method_not_implemented");
        Object[] errArgs = new Object[]{method};
        errMsg = MessageFormat.format(errMsg, errArgs);
        resp.sendError(501, errMsg);
    }

}
```


使用步骤：
1. 定义类继承HTTPServlet
2. 覆写doGet/doPost方法

```java
//在web目录下建立一个login的html文件
@WebServlet(urlPatterns = "/login")
public class servletDemo extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("doGet方法");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("doPost方法");
    }
} 
```
login.html文件
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>index</title>
</head>
<body>
<form action="/login" method="post">
    <input name="username">
    <br>
    <input type="submit" value="提交">
</form>
</body>
</html>
```
然后启动tomcat后，可以1在网页上打开此html，

访问路径后控制台会打印`doGet方法`

点击提交后控制台会打印`doPost方法`


## Servlet相关配置
@WebServlet注解底层
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface WebServlet {
    String name() default "";

    String[] value() default {};
    
    String[] urlPatterns() default {};
    // 是一个数组，可以为一个Servlet定义多个访问路径
    int loadOnStartup() default -1;

    WebInitParam[] initParams() default {};

    boolean asyncSupported() default false;

    String smallIcon() default "";

    String largeIcon() default "";

    String description() default "";

    String displayName() default "";
}
```
1. 一个Servlet可以写多个访问路径
    ```
    @WebServlet({"/login","/login1","/login2"})
    ```
2. urlpattern怎么写
    1. `/xxx`单层路径
    2. `/xxx/xxx`多层路径（目录结构）
    3. `/user/*`通配符写法
    4. `*.do`自定义一个拓展名(不加斜杠)


# HTTP

## 概述
Hiper Text Transfer Protocol 超文本传输协议
- 传输协议：定义了客户端和服务器端通信时，发送数据的格式

客户端给服务器端发送**请求消息**

服务器端解析客户端的**请求消息**，给客户端发送**响应消息**

## 特点
1. 基于TCP/IP的高级协议
2. 默认端口号 80
3. 基于请求/响应模型的：一次请求对应一次响应
4. 无状态的：每次请求之间相互独立，不能交互数据

## 历史版本
- 1.0版本：每一个图片，每一个css等配置文件，都是一个请求，每次都会重复申请连接
- 1.1版本：复用连接


## 请求消息数据格式
Request格式分为四个部分
1. 请求行
    - 请求方式 请求url 请求协议/版本
    
    ```
    GET /login.html HTTP/1.1
    ```
    - 请求方式有七种，常见有两种
        + GET:
            1. 请求参数在请求行中
            2. 请求的url长度有限
            3. 不太安全
        + POST:
            1. 请求参数在请求体中
            2. 请求的url长度没有限制
            3. 相对安全
2. 请求头
    - 请求头名称:请求值（键:值）
3. 请求空行
    - 空行,分隔请求头和请求体
4. 请求体（正文）
    - 封装Post请求消息的请求参数

字符串格式
```
//-------------请求行---------------
GET /login.html HTTP/1.1
//上面为请求行，底下为请求头
//-------------请求头---------------
Host: localhost
//Host:请求主机是localhost
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:69.0) Gecko/20100101 Firefox/69.0
//User-Agent:浏览器告诉服务器，访问使用的浏览器版本信息
//服务端获取该头信息，解决了浏览器的兼容性问题
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
//支持的响应格式
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
//支持的语言环境
Accept-Encoding: gzip, deflate
//支持的解析结构
Content-Type: application/x-www-form-urlencoded
Content-Length: 9
Connection: keep-alive
//keep-alive 表示连接都是可以复用的
Referer: http://localhost/login.html
//告诉服务器，当前请求是从哪里发出的
//作用：1.防盗链：防止其他网页盗取链接 2.统计工作：获取来源处的某些信息，比如统计广告来源
Upgrade-Insecure-Requests: 1
//上面为请求头，底下为参数
//------------参数----------------
username = "输入的内容"
```

