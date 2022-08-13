---
title: Properties
date: 2019-08-21 22:04:01
tags: 
- Java
- 资源配置
categories: 
- 后台
- Java
---

<center>
引言：
Properties配置文件   
</center>

<!--more-->

---

# Properties
> Properties 属性映射：是一种存储键值对的数据结构，在java中用来存储配置信息

特性：
1. 继承自`Hashtable`，每个键及其对应值都是一个字符串
2. 映射可以很容易的存入文件以及从文件中加载
3. 有一个二级表保存默认值
4. 该类也被许多Java类使用，比如获取系统属性时，`System.getProperties`方法就是返回一个`Properties`对象

**Properties集合是唯一一个与IO流相结合的集合**

## 包路径

```
java.util
```

## 常用方法

- Properties集合是一个双列集合，key和value默认都是字符串
- Properties集合有一些操作字符串的特有方法
    ```java
    public Object setProperty(String key,String value)
    //设置键值
    
    public String getProperty(String key)
    //通过key找到value值,用指定的键在此属性列表中搜索属性，相当于Map集合中的get方法
    public String getProperty(String key,String defaultValue)
    //当找不到key值时，会返回设置的默认值
    
    public Set<String> stringPropertyNames()
    //返回此属性列表中的键集，其中该键及其对应值是字符串，相当于Map集合中的keySet方法
    
    ```

```java
Properties prop = new Properties();
prop.setProperty("赵丽颖","160");
prop.setProperty("迪丽热巴","165");
prop.setProperty("古力娜扎","160");
//set只能输入字符串
Set<String> set = prop.stringPropertyNames();
//使用stringPropertyNames方法把集合中的键取出，存储到一个set集合中

//遍历set集合，取出Properties集合的每一个键
for (String s : set) {
    System.out.println(prop.getProperty(s));
}
```

## store方法

```java
public void store(Writer writer,String comments)throws IOException

public void store(OutputStream out,String comments)throws IOException
/*
用来把集合中的临时数据持久化的写入到硬盘中
参数一个是字节流，一个是字符流
String comments是用来解释说明保存的文件是做什么用的，不能使用中文，会产生乱码，默认是Unicode编码，一般使用空字符串
*/
```
步骤：
1. 创建Properties集合对象
2. 使用`setProperty()`方法存入键值对
3. 创建字节/字符流对象
4. 使用Properties集合中的方法`store`，把集合中的临时数据持久化写入到硬盘中存储
5. 释放资源

示例代码
```java
Properties prop = new Properties();
prop.setProperty("username","Jack");
prop.setProperty("password","123");
FileWriter fw = new FileWriter("D:\\a.txt");
prop.store(fw,"Settings");
fw.close();
```
运行后的a.txt文件，会自动加一个时间
```
#Settings
#Fri Jan 17 11:27:04 GMT+08:00 2020
password=123
username=Jack
```

## load方法
```java
public void load(Reader reader)throws IOException

public void load(InputStream inStream)throws IOException

/*参数 Inputstream inStream字节输入流，不能读取含有中文的键值对
Reader reader：字符输入流，能读取含有中文的键值对
*/
```
步骤：
1. 创建Properties集合对象
2. 创建字符/字节输入流
2. 使用Properties集合对象中的方法`load`读取保存键值对的文件
3. 遍历Properties集合


```java
Properties prop = new Properties();
prop.load(new FileReader("D:\\a.txt"));
Set<String> set = prop.stringPropertyNames();
for (String s : set) {
    String value =prop.getProperty(s);
    System.out.println(value);
}
```

注意：
1. 存储键值对的文件中，键与值默认的连接符号可以使用 -，空格（其他符号）
2. 存储键值对的文件中，可以使用#进行注释，被注释的键值对不会再被读取
3. 存储键值对的文件中，键与值默认都是字符串，不用再加引号

## System中的Properties
方法
```java
Properties getProperties()
//获取所有的系统属性，应用必须有权获得属性，否则会抛出一个安全异常

String getProperty(String key)
//获取给定键名对应的系统属性
```

例子
```java
Properties properties = System.getProperties();
Set<String> strings = properties.stringPropertyNames();
for (String string : strings) {
System.out.println(string+"="+properties.getProperty(string));
}
//会打印出所有的系统属性
```

## 缺点
-  某些操作系统没有主目录的概念，所以很难找到一个统一的配置文件位置

可以用`System.out.println(System.getProperties().getProperty("user.home"));`来获取用户主目录
- 关于配置文件的命名没有标准约定，用户安装多个java应用时，很容易发生命名冲突
