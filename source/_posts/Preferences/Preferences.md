---
title: Preferences
date: 2020-01-18 10:39:33
tags: 
- Java
- 资源配置
- Preferences
categories: 
- 后台
- Java
- Preferences
---
<center>
引言：

Preferences配置文件  
</center>

<!--more-->

# Preferences

> Preferences首选项API：存储资源配置，为解决Properties的一些缺点（某些操作系统没有用户主目录的概念，而且大量的java文件有可能会有冲突）而出现的稳定的存储类

Preferences通过类似于包名节点路径的配置`/com/company/app`的存储

特点：
1. Preferences可以有多棵并行的树，常见有两棵树：
    - 用户树：用来配置个性化的设置
    - 系统树：设置应用程序的设置数据等等
2. 每个节点都有一个preferences的对象，也是用键值对来存储数值、字符串、字节数组，但是不能储存可序列化(串行化)的对象（官方认为串行化的格式过于脆弱）

## 获得树根、操作节点
Preferences的每一个结点都是一个Preferences对象，所以我们需要先获得结点
```java
public static Preferences userRoot()
//获取用户树的树根节点

public static Preferences systemRoot()
//获取系统树的树根节点

```java
public abstract Preferences node(String pathname)
/*node方法：可以传入节点的相对路径或者绝对路径名:
    绝对路径名：以 / 开头的路径
    相对路径名：不以 / 开头的路径
*/
```

示例代码
```java
Preferences root = Preferences.userRoot();
//获得用户根节点
Preferences preferences1 = root.node("/demo03/PreferencesTest");
System.out.println(preferences1);
//打印结果：User Preference Node: /demo03/PreferencesTest
```

## 常用方法
获得树根节点后，可以通过如下方法来操作数据

### 存取数据
```java
abstract void put(String key,String value)
abstract void putInt(String key, int value)
//类似的有putDouble等方法
/*
    key：都是String类型
    value：各有不同
*/

abstract String get(String key, String def)
abstract int getInt(String key, int def)
//同理，有类似的getDouble等方法
/*
    key：都是String类型
    def：默认值。没有不带默认值的方法，必须传入一个默认值，为了确保在首选项不可以使用时依旧不会影响程序的运行，提高程序的鲁棒性（健壮性）
*/

String[] keys()
//返回当前节点的所有键
```
示例代码
```java
preferences1.put("username","李白");
String s = preferences1.get("username", "默认值");
System.out.println(s);

String[] keys = preferences1.keys();
for (String key : keys) {
    System.out.println(key+","+preferences1.get(key,"默认值"));
}
```

### 刷新数据
Preferences中，使用异步的方式写入数据，可以使用`flush`方法刷新数据
```java
abstract void flush()
```

### 删除数据
```java
abstract void remove(String key)
//删除指定键对应的值
abstract void removeNode()
//删除当前节点
```
示例代码
```java
preferences1.remove("username");
s = preferences1.get("username", "默认值");
System.out.println(s);//打印结果：默认值
preferences1.removeNode();
preferences1.put("username","李白");//打印报错：Node has been removed.
```

### 导出/导入数据
数据的载体XML文件
```java
static void importPreferences(InputStream is)
//导入一个XML文件的所有数据，注意：导入数据是一个静态方法，可以直接使用类名调用
abstract void exportNode(OutputStream os)
//导出当前节点存储的数据，但是不会导出当前节点的子节点中的数据
abstract void exportSubtree(OutputStream os)
//导出当前节点及子节点的数据
```
导出数据到xml文件
```java
Preferences root = Preferences.userRoot();
Preferences preferences1 = root.node("/demo03/PreferencesTest");
preferences1.exportNode(new FileOutputStream("preferences.xml"));
```
导出的xml文件的格式
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE preferences SYSTEM "http://java.sun.com/dtd/preferences.dtd">
<preferences EXTERNAL_XML_VERSION="1.0">
  <root type="user">
    <map/>
    <node name="demo03">
      <map/>
      <node name="PreferencesTest">
        <map>
          <entry key="username" value="李白"/>
        </map>
      </node>
    </node>
  </root>
</preferences>

```
导入这个文件
```java
Preferences.importPreferences(new FileInputStream("preferences.xml"));
```

## 实现一个判断是否第一次登录的小demo
第一次登录会输出 欢迎你好，
第二次登录会输出 你好，您的上次登录时间是
```java
public class PreferencesTest {
    public static void main(String[] args) throws BackingStoreException, IOException, InvalidPreferencesFormatException {
        Preferences node = Preferences.userRoot().node("/demo03/PreferencesTest");
        boolean isFirstLogin = node.getBoolean("isFirstLogin", true);
        if(isFirstLogin){
            System.out.println("欢迎光临");
            node.putBoolean("isFirstLogin",false);
        }else {
            System.out.println("您好,您的上次登录时间是"+node.get("LoginTime","Error"));
        }
        node.put("LoginTime", String.valueOf(LocalTime.now()));
    }
}
```

