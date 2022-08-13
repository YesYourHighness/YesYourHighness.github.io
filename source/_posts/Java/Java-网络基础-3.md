---
title: Java-网络基础-3
date: 2020-01-31 10:19:40
tags: 
- Java
- Java网络基础
categories: 
- 后台
- Java
---

<center>
引言：

URL和URI
</center>

<!-- more -->


# URI
> URI：统一资源标识符(Uniform Resource Identifier)
，URI只是一个纯粹的语法结构，**唯一的功能就是解析**。
**URL是URI的一个特例**

## URI的组成部分
URI包含用来指定Web资源的字符串的各种组成成分主要由三大部分构成
```
[scheme:]schemeSpecificPart[#fragment]
[方案/协议:]方案具体部分[#片段]
<!-- [...]中为可选 -->
```
包含`scheme:`的URI称为绝对URI，否则称为相对URI

`schemeSpecificPart`不以`/`开头称**不透明**的的

所有的绝对的透明URI（第一个）和所有相对URI（第二个）都是分层的
```
http://horstmann.com/index.html
../../java/net/Socket.html#Socket()
```
一个分层的URI的`schemeSpecificPart`包含以下结构
```
[//authority][path][?query]
[//权威][路径][?查询]
```
而基于服务器的URI，`authority`又有以下形式
```
[user-info@]host[:port]
```
`port`必须是一个整数

所以URI我们可以用以下的方法来读取他们
```java
getScheme()
getSchemeSpecificPart()
getAuthority()
getUserInfo()
getHost()
getPort()
getPath()
getQuery()
getFragment()
```
```java
URI uriAbs = new URI("http://docs.mycompany.com/api/java/net/ServerSocker.html");
URI uriRel = new URI("../../java/net/Socket.html#Socket()");
uriAbs.isAbsolute();// true
uriRel.isAbsolute();// false
uriAbs.getScheme();// http
uriAbs.getSchemeSpecificPart();// 结果是//docs.mycompany.com/api/java/net/ServerSocker.html
    {//以下属于SchemeSpecificPart的组成
        uriAbs.getAuthority();//docs.mycompany.com
            {//以下属于Authority的组成
                uriAbs.getUserInfo();//null
                uriAbs.getHost();//docs.mycompany.com
                uriAbs.getPort();//-1 (端口是未定义的返回-1)
            }
        uriAbs.getPath();///api/java/net/ServerSocker.html
        uriAbs.getQuery();//null
    }
uriAbs.getFragment();//null
uriRel.getFragment();//Socket()
```
## URI类的作用
> 处理绝对标识符和相对标识符（是否包含[scheme:]）

例如：如果存在以下的一个绝对URI，和相对URI，我们可以组合出一个绝对URI
```
绝对URI: 
http://docs.mycompany.com/api/java/net/ServerSocker.html

相对URI：
../../java/net/Socket.html#Socket()

组合后的绝对URI：
http://docs.mycompany.com/api/java/net/ServerSocker.html#Socket()
```
这个过程叫做**解析相对URI**，与此过程相反的过程叫**相对化**
```
基本URI：
http://docs.mycompany.com/api

另一个URI：
http://docs.mycompany.com/api/java/lang/String.html

相对化后的URI：
java/lang/String.html
```
```java
URI uriAbs = new URI("http://docs.mycompany.com/api/java/net/ServerSocker.html");
URI uriRel = new URI("../../java/net/Socket.html#Socket()");

/*
    解析相对URI：
    http://docs.mycompany.com/api/java/net/Socket.html#Socket()
*/
URI uriBase = new URI("http://docs.mycompany.com/api");
URI uriOther = new URI("http://docs.mycompany.com/api/java/lang/String.html");
uriBase.relativize(uriOther);
/*
    相对化：
    java/lang/String.html
*/
```

# URL
> URL：统一资源定位符 (Uniform Resource Locator)，URL可以打开一个到达资源的流，URL类只能作用于那些Java类库知道如何处理的类库，如`http:、https:、ftp:、本地文件系统file:、JAR文件jar:`

## 包路径
```java
java.net
```
## 构造方法
```java
URL(String urlString)
```

## 常用方法
```java
public InputStream openStream()
//获取一个InputStream对象，进而获取URL资源内容
```
## URLConnection
`URLConnection类`比`URL类`有更多的控制
### 包路径
```
java.net
```

### 常用方法
```java
void setDoInput(boolean doInput)
boolean getDoInput()
//若doInput为true，那么用户可以接收来自该URLConnection的输入


void setDoOutput(boolean doOutput)
boolean getDoOutput()
//若doOutput的值为true，用户可以将输出发送到URLConnection

void setIfModifiedSince(long time)
long getIfModifiedSince()
/*
属性IfModifiedSince用于配置该URLConnection对象，使它只获得那些从某个给定时间以来被修改的数据

传入的参数time:是从格林尼治时间开始计算的秒数
*/

void setUseCaches(boolean useCaches)
boolean getUseCaches()
//如果useCaches为true，那么数据可以从本地缓存中得到，缓存由浏览器之外的外部程序提供

void setAllowUserInteraction(boolean allowUserInteraction)
boolean getAllowUserInteraction()
//如果allowUserInteraction为true，那么可以查询用户的口令，口令由浏览器之外的程序提供

void setConnectTimeout(int timeout)
int getConnectTimeout()
//设置或得到连接超时时限

void setReadTimeout(int timeout)
int getReadTimeout()
//设置读取数据的超时时限

void setRequestProperty(String key,String value)
//设置请求头的一个字段

Map<String,List<String>> getRequestProperties()
//返回请求头属性的一个映射表，相同的键对应的所有值被放置在同一个列表中

void connect()
//连接远程资源并获取响应头信息

Map<String,List<String>> getHeaderFields()
//返回响应的一个映射表，相同的键对应的所有值被放置在同一个列表中

String getHeaderFieldKey(int n)
//得到响应头的第n个字段的 键。如果n>=0或者大于响应头字段的总数，返回null

String getHeaderField(int n)
//得到响应头的第n个字段的 值。如果n>=0或者大于响应头字段的总数，返回null

int getContentLength()
//内容长度可以获得返回内容长度，不可获得返回-1

String getContentEncoding()
//获取内容的编码机制

long getDate()
long getExpiration()
long getLastModified()
//获取创建日期，过期日，以及最后一次被修改的日期，这些时间通通都是从格林尼治时间返回的秒数

InputStream getInputStream()
OutputStream getOutputStream()
//返回从资源读取信息或向资源写入信息的流

Object getContent()
//选择适当的内容处理器，以便读取资源数据并将资源数据转换成对象。对text/plain或image/gif之类的标准内容类型没有用处，除非自己安装内容处理器
```



### 操作步骤

```java
//1. 获取URLConnection对象
URLConnection urlConnection = url.openConnection();

//2. 使用各种相关方法来设置任意的请求属性



```