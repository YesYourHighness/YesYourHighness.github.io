---
title: RequestAndResponse
date: 2019-10-11 22:10:11
tags: 
- Request&Response
categories: 
- 后台
- Request&Response
---

<center>
引言：

Request&Response
</center>

<!-- more -->

-------

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
1. **请求行**
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
2. **请求头**
    - 请求头名称:请求值（键:值）
3. **请求空行**
    - 空行,分隔请求头和请求体
4. **请求体**（正文）
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

## 响应消息数据格式
Response也分为四个部分
1. **响应行**
    1. 组成：协议/版本 响应状态码 状态码的描述
    2. 响应状态码：一个三位数字，分为五类，服务器告诉客户端浏览器本次请求和响应的一个状态
        - 1xx ：服务器接收客户端消息但没有接收完成，等待一段时间后发送1xx的一个状态码
        - 2xx ：成功。代表：200(成功)
        - 3xx ：重定向。代表：302(重定向),304(访问缓存)
        - 4xx ：客户端错误。代表：404(请求路径为空)，405(请求方式没有对应的doGet等方法)
        - 5xx ：服务器端错误。代表：500(服务器端出现异常)
2. **响应头**
    1. 格式： 头名称:值
3. **响应空行**
4. **响应体**

字符串格式：
```
HTTP/1.1 200 
//响应行：协议/版本 响应状态码 状态码的描述
Content-Type: text/html;charset=UTF-8
//告诉服务器本次响应体的数据格式以及编码格式
Content-Length: 90
//本次请求的字节数
Date: Tue, 08 Oct 2019 13:39:33 GMT
//日期
Content-disposition:
//服务器告诉客户端以什么打开响应体数据
//值： 1. in-line代表在当前页面打开 2.attachment：以附件形式打开响应体，用于文件下载
//--------以上为响应头----------
//响应体即传送的数据，内容包括我们可以看懂的HTML的内容，还有其他的一些看不懂的内容
```

-------



# Request&Response
重写doGet和doPost方法后，可以接收两个参数Request和Response
```java
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
/**
    1. tomcat服务器会根据url中的资源路径，创建对应的ServletDemo类的对象
    2. tomcat服务器会创建request和response对象，request对象中封装请求消息数据
    3. tomcat将request和response两个对象传递给service方法，并且调用service方法
    4. 我们可以通过操作request对象来获取请求消息数据，通过response对象设置响应消息数据
    5. 服务器在给浏览器做出响应之前，会从response对象中拿出设置的响应消息数据
    
/
```
## 原理
1. request对象和response对象是由服务器创建的
2. request对象来接收消息，response来设置响应消息

# Request


## Request对象继承体系结构
- servletRequest  接口
- HTTPServletRequest  （继承上面的接口）
- org.apache.catalina.connector.RequestFacade  （实现了上面的接口） 


## Request的功能

### 1. 获取请求消息数据
- 获取请求行数据
```
//请求行长这个样子
GET /虚拟目录/Servlet路径?id=xxx HTTP/1.1
```
request的方法：
```java
//1. 获取请求方式：GET
String getMethod()
//2. 获取虚拟目录（重点）
String getContextPath()
//3. 获取Servlet的路径
String getServletPath()
//4. 获取get方式的请求参数
String getQueryString()
//5. 获取请求的URI（重点） 即一次性获取 /虚拟目录/Servlet路径
String getRequestURI()获取到/虚拟目录/Servlet路径
String getRequestURL()获取到http://localhost/虚拟目录/Servlet路径
//6. 获取协议及版本
String getProtocol()
//7. 获取客户机的IP地址
String getRemoteAddr()
```
URI: 统一资源定位符：`/虚拟目录/Servlet路径`

URL：统一资源标识符：`http://localhost/虚拟目录/Servlet路径`

URI的范围更大，他们俩的关系类似于 共和国 和 中华人民共和国 的关系

- 获取请求头数据
方法：
```java
String getHeader(String name)
//1. 通过请求头名称获取请求头的值
Enumeration<String> getHeaderNames()
//2. 获取所有的请求头,返回一个类似于迭代器的类
```
示例代码
```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    //1. 获取所有请求头
    Enumeration<String> headerNames = req.getHeaderNames();
    while (headerNames.hasMoreElements()){
        String name = headerNames.nextElement();
        //2. 据名称获取请求头
        String value = req.getHeader(name);
        System.out.println(name + "---" + value);
    }
}
```
一般不会使用第二种方法，一般的使用情景如下

```java
1. 判断登录平台
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String agent = req.getHeader("user-agent");
    // 此处不区分大小写
    if (agent.contains("Firefox")) {
        System.out.println("火狐登录");
    } else if (agent.contains("Chrome") ){
        System.out.println("谷歌登录");
    }
} 

2. 判断是否盗链
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String referer = req.getHeader("referer");
    if(referer!=null){
        if(referer.contains("/来源路径")){
            //正常
        }
        else {
            //盗链，禁止
        }
    }
}
```


- 获取请求体数据
**只有POST才有请求体**

步骤：
```
//1. 获取流对象
BufferedReader getReader()//获取字符输入流，只能操作字符数据
ServletInputStream getInputStream()//获取字节输入流，可以操作所有类型数据
2. 再从流对象中拿取数据

```
示例代码
```java
@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    BufferedReader br = req.getReader();
    String line = null;
    while ((line=br.readLine())!=null){
        System.out.println(line);
    }
}
```



### 2. Request的其他功能

- 获取请求参数通用方式
```java
//这些都是通用方法，适用于post或get

//1. 获取请求参数通用方式
String getParameter(String name)
//根据参数名称获取参数值
String[] getParameterValues(String name)
//根据参数名称获取参数值的数组 username=2018005761&password=123456
getParameterNames()
//获取所有请求的参数名称
Map<String,String[]> getParameterMap()
//获取所有参数的map集合
```
```java
//中文乱码的问题：
//get方法： Tomcat8.0版本已经解决中文乱码问题
//post方法：在获取参数前，设置请求request的编码
req.setCharacterEncoding("utf-8")
```
- 请求转发
```java
//一种在服务器内部资源跳转的方式
//步骤：
//1. 通过request对象获取请求转发器对象
RequestDispatcher getRequestDispatcher(String path)
//2. 使用RequestDispatcher对象来进行转发
forward(ServletRequest request,ServletResponse response)
```
特点：
1. 浏览器地址不发生变化
2. 不能访问服务器外部的其他资源
3. 转发是一次性的请求
```
//示例代码
// demo1 转发demo
@WebServlet("/login")
public class servletDemo extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        RequestDispatcher dispatcher = req.getRequestDispatcher("/login2");
        dispatcher.forward(req,resp);
    }
}
// demo2 被转发的demo
@WebServlet("/login2")
public class servletDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("我被转发调用了");
    }
}
//这样当访问demo1时，也会自动的访问demo2，而且访问是在同一次的请求完成的
```
- 共享数据
    - 域对象： 一个有作用范围的对象，可以在范围内共享数据
    - request域：代表一次请求
```java
//方法
//1.存储数据
void setAttribute(String name,Obejct);
//2. 通过键获取值
Object getAttribute(String name);
//3. 通过键移除键值对
void removeAttribute(String name)
```
代码示例
```java
@WebServlet("/login")
public class servletDemo extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1. 存储信息
        req.setAttribute("msg","hello");
        //转发
        RequestDispatcher dispatcher = req.getRequestDispatcher("/login2");
        dispatcher.forward(req,resp);
    }
}

@WebServlet("/login2")
public class servletDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("我被转发调用了");
        //2. 获取数据
        System.out.println(req.getAttribute("msg"));
        //3. 删除数据
        req.removeAttribute("msg");
        System.out.println(req.getAttribute("msg"));
    }
}
```

- 获取ServletContext
这里先介绍如何获得ServletContext对象
```java
 ServletContext servletContext = req.getServletContext();
```

# 用户登录案例
需求:
1. 编写login.html登录页面
2. 有username和password两个输入框
3. 使用Druid数据库连接池技术，操作mysql
4. 使用JdbcTemplate技术封装JDBC
4. 登录成功跳转到SuccessServlet展示：登陆成功！用户名，欢迎你
5. 登录失败跳转到FailServlet展示：登陆失败，用户名或密码错误

代码：

1. 创建JavaEE项目
2. 构建html页面
```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
</head>
<body>
<form action="/login" method="post">
    <input type="text" placeholder="请输入用户名" name="username">
    <br>
    <input type="password" placeholder="请输入密码" name="password">
    <br>
    <input type="submit" value="提交">
</form>
</body>
</html>
```
3. 引入jar包到web目录下的WEB_INF目录下
4. 创建数据库环境,在src目录下配置`druid.properties配置文件`
```sql
CREATE DATABASE testlogin;
-- 创建数据库
USE testlogin;
-- 使用此库
CREATE TABLE USER(
	id INT PRIMARY KEY AUTO_INCREMENT,
	username VARCHAR(32) UNIQUE NOT NULL,
	PASSWORD VARCHAR(32) NOT NULL
);
```
配置文件
```
driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/testlogin?serverTimezone=UTC
username=root
password=输入你的密码
initialSize=5
maxActive=10
maxWait=3000
filters=stat
timeBetweenEvictionRunsMillis=60000
minEvictableIdleTimeMillis=300000
validationQuery=SELECT 1
testWhileIdle=true
testOnBorrow=false
testOnReturn=false
poolPreparedStatements=false
maxPoolPreparedStatementPerConnectionSize=200
```
5. src目录下创建包`cn.test.domain`,创建类User,并补充响应的get,set,toString方法
```java
package cn.test.domain;

/**
 * @author BlackKnight
 * 用户的实体类
 */
public class User {
    private int id;
    private String username;
    private String password;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
6. 创建`cn.test.dao`包,创建UserDao，提供login方法
```java
package cn.test.dao;

import cn.test.domain.User;

/**
 * @author BlackKnight
 * 操作数据库中User表的类
 */
public class UserDao {
    /**
     * 登录方法
     * @param loginUser 只要用户名和密码
     * @return user包含用户全部数据
     */
    public User login(User loginUser){
        return null;
    }
}

```
7. 创建工具包`cn.test.util`,创建`JDBCUtils`工具类
```java
package cn.test.util;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

/**
 * @author BlackKnight
 * JDBC工具类 使用Druid连接池
 */
public class JDBCUtils {
    private static DataSource ds;

    static {
        try {
            //1. 加载配置文件
            Properties pro = new Properties();
            //使用ClassLoader加载配置文件
            InputStream is = JDBCUtils.class.getClassLoader().getResourceAsStream("druid.properties");
            pro.load(is);
            //2. 初始化连接池对象
            ds = DruidDataSourceFactory.createDataSource(pro);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    /**
     * 获取连接对象
     */
    public static DataSource getDataSource() {
        return ds;
    }

    /**
     * 获取连接Connection对象
     */
    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }

}

```
8. 完善`UserDao类`
```java
package cn.test.dao;

import cn.test.domain.User;
import cn.test.util.JDBCUtils;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;

/**
 * @author BlackKnight
 * 操作数据库中User表的类
 */
public class UserDao {
    //声明JDBCTemplate对象共用
    private JdbcTemplate template = new JdbcTemplate(JDBCUtils.getDataSource());

    /**
     * 登录方法
     * @param loginUser 只要用户名和密码
     * @return user包含用户全部数据,没有查询到，则返回null
     */
    public User login(User loginUser){
        try {
            //1. 编写sql语句
            String sql = "select * from user where username = ? and password =?";
            //2. 调用query方法
            User user = template.queryForObject(sql,
                    new BeanPropertyRowMapper<User>(User.class),
                    loginUser.getUsername(), loginUser.getPassword());
            return user;
        }catch (DataAccessException e){
            e.printStackTrace();//写入日志
            return null;
        }
    }

}


```
9. 创建包`cn.test.test`,创建UserDao类来检测程序
```java
package cn.test.test;

import cn.test.dao.UserDao;
import cn.test.domain.User;
import org.junit.jupiter.api.Test;

/**
 * @author BlackKnight
 * 测试UserDao类
 */
public class UserDaoTest {
    @Test
    public void testLogin(){
        User loginUser = new User();
        loginUser.setUsername("李白");
        loginUser.setPassword("456");

        UserDao dao = new UserDao();
        User user = dao.login(loginUser);

        System.out.println(user);
    }
}

```
10. 编写`cn.test.web.servlet.LoginServlet`类
```java
package cn.test.web.servlet;

import cn.test.dao.UserDao;
import cn.test.domain.User;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author BlackKnight
 */
@WebServlet("/loginServlet")
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1. 设置编码
        req.setCharacterEncoding("utf-8");
        //2. 获取请求参数
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        //3. 封装user对象
        User loginUser = new User();
        loginUser.setUsername(username);
        loginUser.setPassword(password);
        //4. 调用userDao类操作数据库
        UserDao userDao = new UserDao();
        User user = userDao.login(loginUser);
        //5. 判断user
        if(user==null){
            //登陆失败
            req.getRequestDispatcher("/failServlet").forward(req,resp);
        }else {
            //登录成功
            //存储数据
            req.setAttribute("user",user);
            //转发

            req.getRequestDispatcher("/successServlet").forward(req,resp);
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req,resp);
    }
}

```
11. login.html中form表单的action路径的写法
```java
//虚拟目录 + Servlet的资源路径
<form action="/testLogin/loginServlet" method="post">
```
12.设置`failServlet`与`successServlet`
```java
package cn.test.web.servlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/failServlet")
public class FailServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //给页面写一句话
        //设置编码
        response.setContentType("text/html;charset=utf-8");
        //输出
        response.getWriter().write("登录失败,用户名或密码错误");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}

```
```java
package cn.test.web.servlet;

import cn.test.domain.User;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/successServlet")
public class SuccessServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取request域中共享的对象
        User user = (User) request.getAttribute("user");
        if(user!=null){
            //给页面写一句话
            //设置编码
            response.setContentType("text/html;charset=utf-8");
            //输出
            response.getWriter().write("登录成功,"+user.getUsername()+"你好!");
        }
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}

```

13. 完成BeanUtils工具类，简化数据封装

* BeanUtils是用于封装JavaBean的

- 什么是JavaBean？
    
    JavaBean是标准的Java类
    1. 类必须被public修饰
    2. 必须提供空参的构造器
    3. 成员变量必须使用private修饰
    4. 提供公共的setter和getter方法
- JavaBean的功能：封装数据
- 概念的辨析：成员变量 和 属性。
    
   大部分情况下成员变量和属性指相同的内容，但某些特例下不相同。

    例如：getUsername() -> Username -> username 这就是属性，属性是getter和setter方法截取后的产物
    
    我们可以给一个成员变量的getter方法取另一个名字，此时成员变量和属性就会不一样了
    
    例如：成员变量为gender，而setter方法起名为SetXXX(),此时属性为XXX
- BeanUtils的方法
```java
setProperty()
getProperty()
//上面两个接收三个参数： 1. Bean类名 2. 属性名 3. 变量值，相当于给成员变量赋值
populate()
//接收两个参数，一个是Bean类，一个是map集合
//将map集合的键值对信息，封装到对应的JavaBean对象中
```

使用BeanUtils简化LoginServlet的第二第三步
```java
package cn.test.web.servlet;

import cn.test.dao.UserDao;
import cn.test.domain.User;
import org.apache.commons.beanutils.BeanUtils;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.InvocationTargetException;
import java.util.Map;

/**
 * @author BlackKnight
 */
@WebServlet("/loginServlet")
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1. 设置编码
        req.setCharacterEncoding("utf-8");
        /*
        //2. 获取请求参数

        String username = req.getParameter("username");
        String password = req.getParameter("password");
        //3. 封装user对象
        User loginUser = new User();
        loginUser.setUsername(username);
        loginUser.setPassword(password);
        */
        //2. 获取所有的请求参数
        Map<String, String[]> map = req.getParameterMap();

        //3. 创建User对象
        User loginUser = new User();
        //3.1 使用BeanUtils封装
        try {
            BeanUtils.populate(loginUser,map);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

        //4. 调用userDao类操作数据库
        UserDao userDao = new UserDao();
        User user = userDao.login(loginUser);
        //5. 判断user
        if(user==null){
            //登陆失败
            req.getRequestDispatcher("/failServlet").forward(req,resp);
        }else {
            //登录成功
            //存储数据
            req.setAttribute("user",user);
            //转发

            req.getRequestDispatcher("/successServlet").forward(req,resp);
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req,resp);
    }
}

```



# Response对象
## 功能：设置响应消息
1. 设置响应行
    ```java
    //1. 格式：HTTP/1.1 200 ok
    //2. 设置状态码：
    setStatus(int sc)
    ```
2. 设置响应头
    ```java
    setHeader(String name,String value);

    sendRedirect()//重定向
    ```
3. 设置响应体
    ```java
    //也是通过流的方式设置响应体
    1. 获取输出流
        * 字符输出流 PrintWriter getWriter()
        * 字节输出流 ServletOutputStream getOutputStream()
    2. 将数据输出到客户端浏览器
    ```

### 实例1：重定向
重定向： 
客户端向A请求数据，
**A接到数据后告诉客户端另一个地址B以及重定向状态码302**，
客户端遵循继而转移到B
```java
package cn.test.test;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author BlackKnight
 * 重定向demo
 */
@WebServlet("/responseDemo1")
public class ResponseDemo1 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //访问/responseDemo1,自动跳转到/responseDemo2
        //1. 设置状态码
        response.setStatus(302);
        //2. 设置响应头location
        response.setHeader("location","/testLogin/responseDemo2");
        //由以上可知 重定向的状态码和响应头的键都是固定的302和location
        //所以我们可以使用如下的方法替代上述两种方法,与上述方法二选一即可
        response.sendRedirect("/testLogin/responseDemo2");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}

```
- 转发(forward)的特点：
1. 地址栏路径不变
2. 转发只能访问当前服务器下的资源
3. 转发是一次请求，可以使用request对象来共享数据

- 重定向(redirect)的特点：
1. 地址栏发生变化
2. 重定向可以访问其他站点的资源
3. 重定向是两次请求，不能使用request对象共享数据

- 路径的写法：
1. 路径分类：
    1. 相对路径：通过相对路径不可以确定唯一资源
        - 如`./index.html`
        - 以`.`开头的路径
        - 如`<a href="./responseDemo2">`可以进入同级目录下的其他资源，`../`进入上一级目录,`./`可以省略，写为`<a href="responseDemo2">`
    2. 绝对路径：通过绝对路径可以确定的唯一资源
        - 如：`http://localhost/testLogin/responseDemo1`这就是一个完整的绝对路径，我们也要研究不完整的绝对路径`/testLogin/responseDemo2`
        - 我们可以简单记为以`/`开头的路径
        - 给客户端浏览器访问：需要加虚拟目录(项目的访问路径)
        - 给服务器端访问：不需要加虚拟目录(例如：转发)
2. 动态获取虚拟目录
```
String contextPath = request.getContextPath();
//此方法可以动态的获取路径
response.sendRedirect(contextPath+"/responseDemo2");
```


### 实例2：服务器输出字符数据到客户端浏览器
步骤：
1. 获取字符输出流
2. 输出数据
```java
package cn.test.test;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

/**
 * @author BlackKnight
 * 字符输出到客户端
 */
@WebServlet("/responseDemo2")
public class ResponseDemo2 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 设置编码，默认的浏览器解码是GBK,字符输出流的编码是ISO-8859-1编码
        response.setCharacterEncoding("utf-8");
        // 2. 响应头告诉浏览器使用什么编码
        response.setHeader("content-type","text/html;charset=utf-8");
        // 这个方法其实可以完全替代掉第一步的设置输出流的方法
        // 有一个更简单的方法来设置编码从而省去上述的两步
        response.setContentType("text/html;charset=utf-8");
        // 3. 获取字符输出流
        PrintWriter pw = response.getWriter();
        // 4. 输出数据
        pw.write("你好");
        //因为此PrintWriter是response提供的，
        // 每次请求结束后，response都会被销毁，所以即使是write方法也可以实现print一样的动态刷新页面
        pw.write("<h1>hello</h1>");
        // 输出的数据可以带标签
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}
```

### 实例3：服务器输出字节数据到客户端浏览器

步骤：
1. 获取字节输出流
2. 输出数据
```java
package cn.test.test;

import javax.servlet.Servlet;
import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/responseDemo3")
public class ResponseDemo3 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //0. 设置编码
        response.setContentType("text/html;charset=utf-8");
        //1. 获取字节输出流
        ServletOutputStream sos = response.getOutputStream();
        //2. 输出,getBytes默认解码GBK，和浏览器相同，但我们仍然要设置一下编码
        sos.write("你好".getBytes());
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}

```

### 实例4：验证码
本质是一张图片，为的是防止恶意注册

大多都会动态生成验证码

```java
package cn.test.test;

import javax.imageio.ImageIO;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.IOException;
import java.util.Random;

@WebServlet("/checkCodeServlet")
public class CheckCodeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        int width = 100;
        int height = 50;
        //1. 创建一个对象，在内存中代表一个图片
        BufferedImage image = new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB);

        //2. 美化图片
        //2.1 填充背景色，获取画笔对象
        Graphics g = image.getGraphics();//画笔对象
        g.setColor(Color.pink);//设置画笔颜色
        g.fillRect(0,0,width,height);//填充
        //2.2 画边框
        g.setColor(Color.blue);
        g.drawRect(0,0,width-1,height-1);//画边框
        //2.3 写验证码
        String str = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghigklmnopqrstuvwxyz1234567890";
        Random random = new Random();
        for (int i = 1; i <= 5 ; i++) {
            int index = random.nextInt(str.length());
            char ch = str.charAt(index);
            g.drawString(ch+"",width/5*i,height/2);
        }
        //2.4 画干扰线
        g.setColor(Color.green);
        for (int i = 0; i <5 ; i++) {
            int x1 = random.nextInt(width);
            int y1 = random.nextInt(height);
            int x2 = random.nextInt(width);
            int y2 = random.nextInt(height);
            g.drawLine(x1,y1,x2,y2);
        }

        //3. 将图片输出到页面展示
        ImageIO.write(image,"jpg",response.getOutputStream());
        //三个参数，内存中的图片，图片格式，输出流
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}

```

# ServletContext对象
- 概念：代表整个web应用，可以和程序的容器(服务器)来通信
- 获取：两种方法获取的对象是同一个对象
    1. 通过request对象来获取
        ```java
        getServletContext()
        // 获取ServletContext对象
        ```
    2. 通过HTTPServlet获取
        ```java
        this.getServletContext()
        ```
- 功能
    - 获取MIME类型：
        ```java
        MIME (Multipurpose Internet Mail Extensions) 是描述消息内容类型的因特网标准。
        MIME类型：在互联网通信过程中定义的一种文件数据类型
        格式： 大类型/小类型
        例如：text/html  image/jpeg
        
        //获取：
        String getMimeTyppe(String file)
        ```
    - 域对象：共享数据
    - 获取文件的真实(服务器)路径
```java
package cn.test.test;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author BlackKnight
 * ServletContext的功能
 */
@WebServlet("/servletContextDemo")
public class ServletContextDemo extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        /*
        1. 获取MIME类型
        2. 域对象
        3. 获取文件的真实服务器路径
        */
        //获取ServletContext对象
        ServletContext context = this.getServletContext();
        //1. 获取MIME类型
        String filename  = "a.jpg";
        String mimeType = context.getMimeType(filename);
        System.out.println(mimeType);//image/jpeg
        //2. 域对象
        //域下设置的属性可以共同享用
        context.setAttribute("msg","哈哈哈");
        //context.getAttribute("msg");另一个路径下也可以读取到这个属性值
        context.removeAttribute("msg");
        //ServletContext对象的范围：所有用户所有请求的数据
        //3. 获取文件的真实服务器路径
        String realPath = context.getRealPath("/a.txt");//获取web目录下的文件
        String realPath1 = context.getRealPath("/WEB-INF/b.txt");//获取web-inf目录下的文件
        String realPath2 = context.getRealPath("/WEB-INF/classes/c.txt");//获取src目录下的文件
        System.out.println(realPath);//D:\testLogin\out\artifacts\testLogin_war_exploded\a.txt
        System.out.println(realPath1);//D:\testLogin\out\artifacts\testLogin_war_exploded\WEB-INF\b.txt
        System.out.println(realPath2);//D:\testLogin\out\artifacts\testLogin_war_exploded\WEB-INF\classes\c.txt

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}

```

## 文件下载案例
1. 页面显示超链接
2. 点击超链接后弹出下载提示框
3. 完成图片文件下载

在`login.html`内加一段代码
```html
<a href="/testLogin/downloadDemo?filename=1.jpg">图片1</a>
<a href="/testLogin/downloadDemo?filename=2.jpg">图片2</a>
<!--超链接href属性指向Servlet，传递资源名称filename-->
```
java代码如下
```java
package cn.test.test;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.FileInputStream;
import java.io.IOException;

@WebServlet("/downloadDemo")
public class DownloadDemo extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 获取请求参数，文件名称
        String filename = request.getParameter("filename");
        //2. 使用字节输入流加载文件进内存
        //2.1 找到文件的服务器路径
        ServletContext context = this.getServletContext();
        String realPath = context.getRealPath("/img/" + filename);
        //2.2 使用字节流关联
        FileInputStream fis = new FileInputStream(realPath);
        //3. 设置response的响应头
        //3.1 设置响应头类型
        String mimeType = context.getMimeType(filename);//获取文件的MIME类型
        response.setHeader("content-type", mimeType);
        //3.2 设置响应头打开方式
        response.setHeader("content-disposition", "attachment;filename=" + filename);
        //4. 将输入流数据写出到输出流中
        ServletOutputStream sos = response.getOutputStream();
        byte[] buff = new byte[1024 * 8];
        int len = 0;
        while ((len = fis.read(buff)) != -1) {
            sos.write(buff, 0, len);
        }
        fis.close();

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}

```
