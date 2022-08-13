---
title: AJAX
date: 2019-10-31 14:37:43
tags: 
- AJAX
categories: 
- 前端
- AJAX
---

 <center>
引言：

Ajax：一种更新部分网页的技术

</center>

<!--more-->


# AJAX
> 概念：ASynchronous JavaScript And XML 异步的JavaScript和XML


## 异步与同步
客户端和服务器端相互通信的基础上
> 同步：客户端必须等待服务器端的响应，在等待期间客户端客户端不能做其他事情

> 异步：客户端发送请求后，无需等待服务器端响应，可以执行其他操作

Ajax可以实现对网页部分的局部更新，无需更新整个页面

- 异步的优点：
    1. 提升效率，节省资源
    2. 可以立即返回初步的结果
    3. 可以等多次调用的结果出来后在统一返回一次结果的集合，提高响应效率
- 异步的缺点
    1. 异步的操作难以控制，处理困难
    2. 不容易捕获异常
- 异步的使用场景：
    1. IO流操作比较耗时，使用异步可以提高响应速度
    2. 不涉及共有资源时

## JS源码实现Ajax
原生js实现(了解)
```js
function fun(){
    var xmlhttp;
    if (window.XMLHttpRequest) {
        //code for IE7+ , Firefox , Opera, Safari
        xmlhttp = new XMLHttpRequest();
    } else {
        //code for IE6,IE5
        xmlhttp = new ActiveXObject("Microsoft.XMLHTTP")
    }
    //建立连接
    /*
    参数：
    1. 请求方式
    2. 请求路径URL
    3. 布尔值，true代表异步，false代表同步
     */
    xmlhttp.open("GET", "ajaxServlet?username=tom", true);
    //发送
    /*
    GET方式的参数在url后拼接，send方法空参
    POST方法的参数在send内部，url后不需拼接
     */
    xmlhttp.send();
    //4.xmlhttp.responseText接收并处理来自服务器的响应结果
    //但我们需要等待服务器响应成功后再获取
    //当xmlhttp对象就绪状态改变时，触发事件onreadystatechange
    xmlhttp.onreadystatechange=function () {
        /*
        readystate有四种状态：
        0： 请求未初始化
        1：服务器连接已建立
        2：请求已接收
        3：请求处理中
        4：请求已完成，且响应已就绪
         */
        // 判断就绪状态是否为4且响应状态码是否为200
        if(xmlhttp.readyState ==4&& xmlhttp.status==200){
            document.getElementById("div1").innerHTML=xmlhttp.responseText;
        }
    }
}

```
servlet:
```java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/ajaxServlet")
public class ajaxServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        System.out.println(username);
        response.getWriter().write("the request has received");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}

```

## Jquery实现Ajax
1. `$.ajax({键:值})`
```js
//使用$.ajax()发送异步请求
$.ajax({
    url: "ajaxServlet",//请求路径
    type: "POST",//请求方式
    data:{//请求参数
        "username":"jack",
        "age":23
    },
    //响应成功后的回调函数
    success:function (data) {
        alert(data);
    },
    error:function () {
        alert("出错了")
    },
    dataType:"text"//设置接收到的响应数据的格式
})
```  


2. `$.get `:发送get请求

语法：`$.get(url,[data],[callback],[type])`

参数：
1. url：请求路径
2. data：请求参数（可选）
3. callback：回调函数（可选）
4. type：响应结果类型（可选）
```js
$.get("ajaxServlet",{"username":"jack"})
```
3. `$.post`：发送post请求
```
$.post("ajaxServlet",{"username":"jack"})
```
