---
title: 跨域
date: 2020-08-22 16:33:39
tags:
- CORS
categories:
- 其他
---

<center>
引言：HTTP访问控制，CORS跨域资源分享；什么是跨域，以及预检请求等
</center>

<!-- more -->

# HTTP访问控制

## 发现

在web开发中，会配置拦截器，在拦截器中，有这么一个配置：

```java
if (request.getMethod().equalsIgnoreCase("options")) {
    return true;
}
```

`OPTIONS`请求，这个`OPTIONS`请求到底是什么？



还有跨域到底是什么？

## 跨域是什么？

首先我们需要了解一些网络这方面的专业术语

### Origin 同源

> Web内容的源由用于访问它的[URL](https://developer.mozilla.org/en-US/docs/Glossary/URL) 的**方案(协议)，主机(域名)和端口**定义。
>
> 只有当方案，主机和端口都匹配时，两个对象具有相同的起源。

- 方案（scheme）：就是协议，例如HTTP、HTTPS、FTP等等协议

- 主机（host）：也就是主机的IP地址，如果有域名代表域名

- 端口（port）：端口，共有65536个，0-65535，0-1023是周知端口，[端口详细链接](https://baike.baidu.com/item/%E7%AB%AF%E5%8F%A3/103505?fr=aladdin)

同源的例子：

```
http://example.com/app1/index.html
http://example.com/app2/index.html
-------------------------------------------
http://Example.com:80
http://example.com
```

他们有相同的协议和主机，端口都是默认的80端口

不同源的例子：

```

http://example.com/app1
https://example.com/app2	different schemes
-------------------------------------------------------
http://example.com
http://www.example.com
http://myapp.example.com	different hosts
-------------------------------------------------------
http://example.com
http://example.com:8080
```

他们都是不同源的

#### 同源不同源有什么作用？

同源概念来自**同源策略**

> 同源策略的作用：同源策略控制不同源之间的交互

某些操作，仅只限于同源内容才能操作，这种操作就叫**跨源网络访问**

## 跨源网络访问

相关链接：[CORS](https://developer.mozilla.org/zh-CN/docs/Glossary/CORS)

> **CORS** （Cross-Origin Resource Sharing，跨域资源共享）是一个系统，它由一系列传输的[HTTP头](https://developer.mozilla.org/en-US/docs/Glossary/Header)组成，这些HTTP头决定浏览器是否阻止前端 JavaScript 代码获取跨域请求的响应。
>
> [同源安全策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy) 默认阻止“跨域”获取资源。但是 CORS 给了web服务器这样的权限，即服务器可以选择，允许跨域请求访问到它们的资源。

分为三类：

- 跨域**写操作**（Cross-origin writes）一般是被允许的。例如链接（links），重定向以及表单提交。特定少数的HTTP请求需要添加 [preflight](https://developer.mozilla.org/zh-CN/docs/HTTP/Access_control_CORS#Preflighted_requests)。
- 跨域**资源嵌入**（Cross-origin embedding）一般是被允许（后面会举例说明）。
- 跨域**读操作**（Cross-origin reads）*一般是不被允许的*，但常可以通过内嵌资源来巧妙的进行读取访问。例如，你可以读取嵌入图片的高度和宽度，调用内嵌脚本的方法

以下是可能嵌入跨源的资源的一些示例：

- `<script src="..."></script>` 标签嵌入跨域脚本。语法错误信息只能被同源脚本中捕捉到。
- `<link rel="stylesheet" href="...">` 标签嵌入CSS。由于CSS的[松散的语法规则](http://scarybeastsecurity.blogspot.dk/2009/12/generic-cross-browser-cross-domain.html)，CSS的跨域需要一个设置正确的 HTTP 头部 `Content-Type` 。不同浏览器有不同的限制： [IE](http://msdn.microsoft.com/zh-CN/library/ie/gg622939(v=vs.85).aspx), [Firefox](http://www.mozilla.org/security/announce/2010/mfsa2010-46.html), [Chrome](http://code.google.com/p/chromium/issues/detail?id=9877), [Safari](http://support.apple.com/kb/HT4070) (跳至CVE-2010-0051)部分 和 [Opera](http://www.opera.com/support/kb/view/943/)。
- 通过 `<img>`展示的图片。支持的图片格式包括PNG,JPEG,GIF,BMP,SVG,...
- 通过 `<video>` 和 `</video>`播放的多媒体资源。
- 通过 `<object>`、 [``](https://developer.mozilla.org/zh-CN/docs/HTML/Element/embed) 和 `<applet>` 嵌入的插件。
- 通过 `@font-face` 引入的字体。一些浏览器允许跨域字体（ cross-origin fonts），一些需要同源字体（same-origin fonts）。
- 通过 `<iframe>` 载入的任何资源。站点可以使用 [X-Frame-Options](https://developer.mozilla.org/zh-CN/docs/HTTP/X-Frame-Options) 消息头来阻止这种形式的跨域交互。

## 预检请求

跨域资源共享标准新增了一组 HTTP 首部字段，允许服务器声明哪些源站通过浏览器有权限访问哪些资源

CORS的头有下面这些：

```shell
Access-Control-Allow-Origin
# 指示请求的资源能共享给哪些域。
Access-Control-Allow-Credentials
# 指示当请求的凭证标记为 true 时，是否响应该请求。
Access-Control-Allow-Headers
# 用在对预请求的响应中，指示实际的请求中可以使用哪些 HTTP 头。
Access-Control-Allow-Methods
# 指定对预请求的响应中，哪些 HTTP 方法允许访问请求的资源。
Access-Control-Expose-Headers
# 指示哪些 HTTP 头的名称能在响应中列出。
Access-Control-Max-Age
# 指示预请求的结果能被缓存多久。
Access-Control-Request-Headers
# 用于发起一个预请求，告知服务器正式请求会使用那些 HTTP 头。
Access-Control-Request-Method
# 用于发起一个预请求，告知服务器正式请求会使用哪一种 HTTP 请求方法。
Origin
# 指示获取资源的请求是从什么域发起的。
```



规范要求，对那些**可能对服务器数据产生副作用**的 HTTP 请求方法（特别是 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET) 以外的 HTTP 请求，或者搭配某些 MIME 类型的 [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST) 请求），

浏览器必须首先使用 [`OPTIONS`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS) 方法发起一个**预检请求（preflight request）**，从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。如果预检请求得不到回答，那么实际的HTTP请求将不会发送



>  可能对服务器数据产生副作用的HTTP请求

阅读上文，我们注意到这句话，那么什么才是可能对服务器数据产生副作用的HTTP请求？

### 简单请求

一些请求不会触发CORS预检请求，这类请求称为简单请求（并不是规范术语）

符合下面所有条件，就称为简单请求：

- 使用`GET`、`HEAD`、`POST`方法

- HTTP请求头限制这几种字段（不得人为设置该集合之外的其他首部字段）：`Accept、Accept-Language、Content-Language、Content-Type（需要注意额外的限制，safari对这个字段的值做了限制）、DPR、Downlink、Save-Data、Viewport-Width、Width`

- `Content-type`只能取：`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

- 请求中的任意`XMLHttpRequestUpload`对象均没有注册任何事件监听器；`XMLHttpRequestUpload `对象可以使用 `XMLHttpRequest.upload`属性访问。

- 请求中没有使用 `ReadableStream`对象。



### 预检请求

也就是非简单请求，会在正式的HTTP通信之前增加一次HTTP请求

我们平常的web项目中，会有：

- 有`PUT`和`DELETE`请求
- 有`TOKEN`等自定义的头部串
- `content-type`类型为`application/json`

所以我们web项目中的大部分请求全是需要预检的请求



## 小结

本篇说明了以下内容：

### 什么是跨域？

不同源即为跨域，不同源就是协议、主机、端口任意一个不相同

### 什么是预检请求

在处理非简单请求时，会发送预检请求，请求的方法为`OPTIONS`，如果预检请求失败，那么真正的HTTP请求将不会发送