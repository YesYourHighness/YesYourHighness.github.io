---
title: Java-File
date: 2019-08-20 21:44:41
tags: 
- Java
categories: 
- Java
---

<center>
引言：用Java进行对文件的操作使用File类
</center>
<!--more-->

-----

# File类

文件和目录路径名的抽象表示形式

> Java把电脑中的文件和文件夹封装为了一个File类，我们可以使用File类对文件和文件夹进行操作

注意：

- File类与操作系统无关，任何操作系统都可以使用这个类

- 用于文件和目录的创建、查找和删除等操作

## 分隔符的使用
```java
static String pathSeparator 
//与系统有关的路径分隔符，为了方便，它被表示为一个字符串。 
static char pathSeparatorChar 
//与系统有关的路径分隔符。 
static String separator 
//与系统有关的默认名称分隔符，为了方便，它被表示为一个字符串。 
static char separatorChar 
//与系统有关的默认名称分隔符。 

```
示例代码
```java
String pathSeparator = File.pathSeparator;
System.out.println(pathSeparator);
//路径分隔符是 windows是分号; Linux是冒号:

String separator = File.separator;
System.out.println(separator);
//文件名称分隔符 windows反斜杠\ Linux正斜杠/

//布置在服务器上，路径不能写死，因为服务器有可能是windows也有可能是Linux
/**
 * C:\a\a.txt   windows
 * C:/a/a.txt   Linux
 * 我们要写成以下这个样子
 * "C:"+File.separator+"a"+File.separator+"a.txt"
 */
```

## 绝对路径与相对路径
注意事项：

1. 路径不区分大小写
2. 路径中的文件名称分隔符，windows使用反斜杠要写两个表示一个反斜杠


## 构造方法
```java
File(String pathname) 
//通过将给定路径名字符串转换为抽象路径名来创建一个新 File 实例。
//参数可以是以文件结尾，也可以是以文件夹结尾
//路径可以是相对路径也可以是绝对路径
//路径可以是存在的也可以是不存在的
//创建File对象只是把字符串路径封装为File对象不考虑路径的真假情况

File(String parent, String child) 
//根据 parent 抽象路径名和 child 路径名字符串创建一个新 File 实例。 
//好处:父路径和子路径可以单独书写，使用起来十分灵活，父路径和子路径都可以变化

File(File parent, String child) 
//与第二个类似，但是第一个参数是File类
//好处：父路径是File类型，可以使用File类的方法对路径进行一些操作，再使用路径创建对象
```
第一个构造
```java
File f1 = new File("D:\\百度");
System.out.println(f1);
//打印出D:\百度，说明重写了toString方法
```
第二个构造
```java
public static void main(String[] args) {
    show2("D:\\","百度");
} 

private static void show2(String parent, String child) {
    File f2 = new File(parent,child);
    System.out.println(f2);
}
```
第三个构造方法
```java
File parent = new File("C:\\");
File file = new File(parent,"hello.java");
System.out.println(file);
//打印出C:\hello.java
```
## 常用方法
- 获取功能的方法
```java
public String getAbsolutePath()
//返回此抽象路径名的绝对路径名字符串。

public File getAbsoluteFile()
//返回此抽象路径名的绝对路径名形式

public String getName()
//返回由此抽象路径名表示的文件或目录的名称

public long length()
//返回由此抽象路径名表示的文件的大小,以字节为单位
//如果此路径名表示一个文件夹，则返回值是不确定的
//如果路径不存在则返回0
```
示例代码
```java
File f1 = new File("C:\\a.txt");
File f2 = new File("a.txt");
String str = f1.getAbsolutePath();
String str2 = f2.getAbsolutePath();

System.out.println(str);
//打印出绝对路径C:\a.txt
System.out.println(str2);
//打印出绝对路径D:\basic-code\a.txt
//默认会打印出当前路径下的绝对路径

System.out.println(f1.getPath());
//打印出绝对路径C:\a.txt
System.out.println(f2.getPath());
//打印出相对路径a.txt

System.out.println(f1);
//C:\a.txt
System.out.println(f1.toString());
//C:\a.txt,其实调用的就是toString方法

System.out.println(f1.getName());
//a.txt
System.out.println(f2.getName());
//a.txt，获取要么是文件要么是文件夹

System.out.println(f1.length());
//0，文件不存在返回0
System.out.println(f2.length());
//0
```

- 判断功能的方法
```java
public boolean exists()
//测试此抽象路径名表示的文件或目录是否存在。 

public boolean isDirectory()
//测试此抽象路径名表示的文件是否是一个目录。 
//路径不存在返回false

public boolean isFile()
//测试此抽象路径名表示的文件是否是一个标准文件
//路径不存在返回false
```
示例代码
```java
File f1 = new File("C:\\a.txt");
File f2 = new File("a.txt");

System.out.println(f1.exists());
System.out.println(f2.exists());

System.out.println(f1.isFile());
System.out.println(f2.isFile());

System.out.println(f1.isDirectory());
System.out.println(f2.isDirectory());
```

- 创建删除功能
```java
public boolean createNewFile()throws IOException
//当且仅当不存在具有此抽象路径名指定名称的文件时，不可分地创建一个新的空文件
//只能创建文件，并且路径必须存在

public boolean delete()
//删除此抽象路径名表示的文件或目录。如果此路径名表示一个目录，则该目录必须为空才能删除。 
//delete不走回收站，会直接删除

public boolean mkdir()
//创建此抽象路径名指定的目录,单级文件夹

public boolean mkdirs()
//创建此抽象路径名指定的目录，包括所有必需但不存在的父目录。
//注意，此操作失败时也可能已经成功地创建了一部分必需的父目录。 
```
示例代码
```java
File f1 = new File("D:\\a.md");
//在此声明你想要创建的文件,给文件起名
f1.createNewFile();
//文件存在返回false
f1.delete();


File f2 = new File("D:\\a");
f2.mkdir();
//只能创建单级文件夹
f2.delete();

File f3 = new File("D:\\a\\b");
f3.mkdirs();
//创建多级文件夹
f3.delete();
```
- 遍历文件夹功能

```java
public String[] list()
//返回一个字符串数组，表示该目录中的所有子文件或目录

public File[] listFiles()
//返回一个File数组，表示该File目录中的所有子文件或目录

//注意：
//如果路径不存在或不是一个文件夹，会抛出空指针异常
```
示例代码

```java
File f1 =new File("C:\\");
//路径不存在会报错和空指针异常
String[] arr = f1.list();
for (String s : arr) {
    System.out.println(s);
}
//隐藏文件也会看到
```

## 循环遍历多级目录
```java
public static void main(String[] args) {
    File file = new File("D:\\blog");
    showAllFiles(file);
}

private static void showAllFiles(File file) {
    System.out.println(file);
    File[] f = file.listFiles();
    for (File s : f) {
        if(s.isDirectory()){
            showAllFiles(s);
        }
        else {
            System.out.println(s+":"+s.length());
        }
    }
}
```

## 搜索文件
```java
public static void main(String[] args) {
    File file = new File("D:\\BaiduNetdiskDownload");
    showAllFiles(file);
}

private static void showAllFiles(File file) {
    File[] f = file.listFiles();
    for (File s : f) {
        if (s.isDirectory()) {
            showAllFiles(s);
        } else {
            if (s.getName().endsWith(".mp4"))
            //搜索所有以mp4为结尾的文件
            {
                System.out.println(s.getName());
            }
        }
    }
}
```

## 文件过滤器

```java
public File[] listFiles(FileFilter filter)
//java.io.FileFilter是一个接口，用于抽象路径名的过滤
//boolean accept(File pathname) 含有这个方法，可以用来过滤
//参数是一个ListFiles方法遍历目录得到的每一个文件对象


public File[] listFiles(FilenameFilter filter)
//java.io.FilenameFilter实现此类实例可用于过滤器的文件名
//boolean accept(File dir,String name)
//其构造函数有两个参数，
//第一个是构造方法中传递的被遍历的目录，
//第二个是ListFiles遍历目录获取的每一个文件/文件夹的名称

//注意：这两个接口都没有实现类
//需要我们写实现类，重写过滤的方法accept，重写过滤的方法
```
FileFilter实现类
```java
public class FileFilterImpl implements FileFilter {
    @Override
    public boolean accept(File pathname) {
        //定义过滤规则
        //true将该文件返回到调用处，false将不会返回
        if (pathname.isDirectory()) {return true;}
        //如果是文件夹也返回
        return pathname.getName().toLowerCase().endsWith("mp4");
    }
}
```
main函数实现代码
```java
public static void main(String[] args) {
    File file = new File("D:\\BaiduNetdiskDownload");
    showAllFiles(file);
}

private static void showAllFiles(File file) {
    File[] f = file.listFiles(new FileFilterImpl());
    //传递过滤器对象
    for (File s : f) {
        if (s.isDirectory()) {
            showAllFiles(s);
        } else {
                System.out.println(s.getName());
        }
    }
}
```

FilenameFilter内部类
```java
public static void main(String[] args) {
    File file = new File("D:\\BaiduNetdiskDownload");
    showAllFiles(file);
}

private static void showAllFiles(File file) {
    File[] f = file.listFiles(new FilenameFilter() {
        @Override
        public boolean accept(File dir,String name) {
            //过滤规则
            return new File(dir,name).isDirectory()||name.endsWith(".mp4");
        }
    });
    //传递过滤器对象
    for (File s : f) {
        if (s.isDirectory()) {
            showAllFiles(s);
        } else {
            System.out.println(s.getName());
        }
    }
}
```