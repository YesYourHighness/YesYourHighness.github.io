---
title: Java-网络基础-2
date: 2019-08-24 11:40:31
tags: 
- Java
- Java网络基础
categories: 
- 后台
- Java
---

<center>
引言：

网络基础入门

文件上传

B\S通信
</center>

<!--more-->

----

# 文件上传

明确：
1. 数据源
2. 目的地


## 客服端


```java
FileInputStream fis = new FileInputStream("D:\\a.txt");
//1. 读取本地文件，创建本地字节输入流 FileInputStream 对象，构造函数绑定数据源
Socket socket = new Socket("127.0.0.1",8888);
//2. 创建客户端 Socket 对象，构造方法中绑定服务器的IP地址和端口号
OutputStream os = socket.getOutputStream();
//3. 使用`Socket`中的方法 getOutputStream ，获取网络字节输出流 OutputStream 对象
int len = 0;
byte[] bytes = new byte[1024];
while ((len = fis.read(bytes))!=-1){
//4. 使用本地字节输入流`FileInputStream`对象中的方法`read`，获取本地文件
    os.write(bytes,0,len);
    //5. 使用网络字节输出流`OutputStream`对象的write方法，把读取到的文件上传到服务器
}

//socket.shutdownOutput();
//此项加上，终止程序死循环
InputStream is = socket.getInputStream();
//6. 使用`Socket`中的方法`getInputStream`获取网络字节输入流
while ((len = is.read(bytes))!=-1){
//7. 使用网络字节输入流`InputStream`对象中的方法`read`读取服务器回写的数据   
    System.out.println(new String(bytes,0,len));
    //8. 释放资源（`FileInputStream`,`Socket`）
}
fis.close();
socket.close();
```

## 服务器端
明确：
1. 数据源：客户上传的文件
2. 目的地：服务器的硬盘的某一个位置

```java
ServerSocket server = new ServerSocket(8888);
//1. 创建一个服务器ServerSocket对象，和系统要指定的端口
Socket socket = server.accept();
//2. 使用ServerSocket对象中的方法accept，获取到请求的客户端Socket对象
InputStream is = socket.getInputStream();
//3. 使用Socket对象的getInputStream获取网络字节输入流InputStream对象
File file = new File("D:\\upload");
if (!file.exists()) {
    file.mkdirs();
}
//4. 判断目的地文件夹是否存在
FileOutputStream fos = new FileOutputStream(file + "a.txt");
//5. 创建一个本地的字节输出流FileOutputStream对象，构造方法绑定输出目的地
int len = 0;
byte[] bytes = new byte[1024];
while ((len = is.read(bytes)) != -1) {
//6. 使用网络字节输入流InputStream的read方法读取上传的文件
    fos.write(bytes, 0, len);
    //7. 使用本地字节输出流FileOutputStream对象中的方法write，把读取的文件保存着服务器硬盘上
}
socket.getOutputStream().write("上传成功".getBytes());
//8. 使用Socket对象中的方法getOutputStream，获取网络字节输出流OutputStream对象
//9. 使用网络字节输出流OutputStream对象中的write方法给客户端回写数据“上传成功”
fos.close();
socket.close();
server.close();
//10. 释放资源（FileOutputStream,Socket,ServerSocket）
```


运行服务器端和客户端，发现服务器端与客户端并没有停止下来，但是文件已经上传了

其实是因为FileInputStream中的read方法，当其没有可输入时，此方法会阻塞

**read方法永远也读取不到文件的结束标记**

因为我们没有写结束标记，读取不到就会进入阻塞状态，进入死循环

解决方法：

上传完文件，给服务器写一个结束标记，`void shutdownOutput()`，禁用此套接字的输出流,任何以前写的数据都会被发送并且文件会正常结束


## 文件上传优化

```java
ServerSocket server = new ServerSocket(8888);
//让服务器一直处于监听状态（死循环accept方法）
//有一个客户端上传文件，让服务器保存一个文件
while (true) {
    Socket socket = server.accept();
    //使用多线程技术，提高程序效率
    //有一个客户端上传文件就开启一个线程，完成文件的上传
    new Thread(new Runnable() {
        //开启线程完成上传
        @Override
        public void run() {
            try{
                InputStream is = socket.getInputStream();
                File file = new File("D:\\upload");
                if (!file.exists()) {
                    file.mkdirs();
                }
                //文件名称优化
                //自定义一个文件的命名规则：防止同名的文件被覆盖
                //规则：域名+毫秒值+随机数
                String fileName = "域名" + System.currentTimeMillis() + new Random().nextInt(9999) + ".txt";
                FileOutputStream fos = new FileOutputStream(file + "\\" + fileName);
                int len = 0;
                byte[] bytes = new byte[1024];
                while ((len = is.read(bytes)) != -1) {
                    fos.write(bytes, 0, len);
                }
                socket.getOutputStream().write("上传成功".getBytes());
                fos.close();
                socket.close();
            }
            catch (IOException e){
                System.out.println(e);
            }
        }
    }).start();
}
//server.close();
//服务器不需要关闭
```

# 模拟B\S服务器

客户端不再是Java程序，而是一个web页面

服务器端
```java
//创建一个服务器
ServerSocket server = new ServerSocket(8080);
//获取到请求的浏览器
Socket socket = server.accept();
InputStream is = socket.getInputStream();
byte[] bytes = new byte[1024];
int len = 0;
while ((len = is.read(bytes))!=-1){
    System.out.println(new String(bytes,0,len));
}
```
网页地址输入
```
http://127.0.0.1:8080/项目名目录地址/index.html
```
服务端打印
```
GET /day04-code/src/index.html HTTP/1.1
Host: 127.0.0.1:8080
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Sec-Fetch-Site: none
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
```
服务器端要响应用户的信息，回应一个html页面，我们需要读取index文件必须知道这个文件的地址，二者地址就是请求信息的第一行
```
GET /day04-code/src/index.html HTTP/1.1
```
我们可以使用`BufferedReader`的`readLine`读取该一行，用String类的`split(" ")`切割字符串，得到
```
/day04-code/src/index.html
```
再使用String类的`subString(1)`获取html文件的路径
```
day04-code/src/index.html
```
这样就得到了路径

服务器创建一个本地的字节输入流，根据获取到的文件路径，获取html文件,并写入固定三行代码

```java
//写入HTTP协议请求响应头
os.write("HTTP/1.1 200 OK\r\n".getBytes());
os.write("Content-Type:text/html\r\n".getBytes());
//必须要写入空行，否则浏览器不解析
os.write("\r\n".getBytes());
```
服务器端使用网络字节输出流把读取到的文件写到客户端

实现代码：
```java
//创建一个服务器
ServerSocket server = new ServerSocket(8080);
while (true) {
    Socket socket = server.accept();
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                //获取到请求的浏览器
                InputStream is = socket.getInputStream();
                BufferedReader br = new BufferedReader(new InputStreamReader(is));
                //只读取一行
                String line = br.readLine();
                //处理地址
                String[] arr = line.split(" ");
                String htmlPath = arr[1].substring(1);
                //创建一个本地的字节输入流
                FileInputStream fis = new FileInputStream(htmlPath);
                OutputStream os = socket.getOutputStream();
                //写入HTTP协议请求响应头
                os.write("HTTP/1.1 200 OK\r\n".getBytes());
                os.write("Content-Type:text/html\r\n".getBytes());
                //必须要写入空行，否则浏览器不解析
                os.write("\r\n".getBytes());
                //一读一写复制文件
                int len = 0;
                byte[] bytes = new byte[1024];
                while ((len = fis.read(bytes)) != -1) {
                    os.write(bytes, 0, len);
                }
                fis.close();
                socket.close();
            } catch (IOException e) {
                System.out.println(e);
            }
        }
    }).start();
}
```