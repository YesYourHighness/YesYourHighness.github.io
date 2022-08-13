---
title: XML
date: 2019-09-03 17:11:53
tags: 
- XML
categories: 
- 后台
- XML
---

<center>
引言：

XML
</center>


<!-- more -->

-----

# XML
XML也是一门标记语言


## 了解
- 概念
Extensible Markup Language 可扩展标记语言

可扩展：标签都是自定义的

- XML的发展历程

w3c(万维网联盟)创建了这两个语言，最早出现了HTML，由于浏览器之间的竞争，HTML发展的缓慢，w3c一度想要放弃HTML，想要用XML来替代HTML，当时XML比HTML语法严谨外没有功能拓展，也不受青睐

最后XML走了另一条路，成为了配置文件的内容

例如properties如此存数据
```
name = zhangsan
age =23
gender = nan
```
但是XML更加清晰
```
<user>
    <name>zhangsan</name>
    <age>age</age>
</user>
```
- XML与HTML的区别
1. XML标签自定义，HTML标签预定义
2. XML语法严格，HTML松散
3. XML存储数据，HTML展示数据



- 功能
1. 配置文件
2. 在网络中传输



## 语法

### 基本语法
- 文档后缀名` .xml`
- 文档声明必须写且必须顶第一行写
- 必须有根标签
- 属性必须使用引号引起，单双都可以
- 标签随意定义，但必须闭合
- 标签区分大小写

### 快速入门
```xml
<?xml version='1.0'?>

<users>
	<user id='1'>
		<name>zhangsan</name>
		<age>23</age>
		<gender>male</gender>
	</user>
	<user id='1'>
		<name>lisi</name>
		<age>20</age>
		<gender>female</gender>
	</user>
</users>
```


### 组成部分
1. 文档声明
    1. 格式`<?xml 属性列表 ?>`
    2. 属性列表
        - version 版本号,必须的属性，写1.0即可
        - encoding 编码方式，告知解析引擎当前文档使用的字符集，默认`ISO-8859-1`
        - standalone 是否依赖其他文件,取值有yes和no
2. 指令(了解)
    ```xml
    <?xml-stylesheet type="text/css" href="a.css"?>
    ```
    
    ```css
    name{
    color: red;
    }
    ```
3. 标签

命名规则
    1. 名称不能以数字或符号开头
    2. 名称不能以xml开始
    3. 名称不能有空格

4. 属性

    属性必须有引号    

    **id属性值唯一**

5. 文本内容

    代码内容使用CDATA区，该区的代码原样展示
    ```
     <![CDATA[
        代码写在这里
    ]]>
    ```
### 约束

约定XML文档的书写规则，用来给框架程序解析，也用来给程序员约定书写规范

作为框架的使用者的我们

只需要

1. 简单的读懂
2. 会引用

分类：
- DTD：一种简单的约束技术
- Schema：复杂的约束技术


#### DTD

引入DTD文档到XML文档中

分两种
- 内部dtd ：将约束规则定义在xml文档中
- 外部dtd ：将约束的规则定义在外部的dtd文件中
    - 本地：`<!DOCTYPE 根标签名 SYSTEM "dtd文件的位置">`
    - 网络：`<!DOCTYPE 根标签名 PUBLIC "dtd文件名字" "dtd文件的位置URL">`

dtd文档
```
<!--dtd文档-->
<!ELEMENT students (student*) >
<!--正则表达式，* 表示写0次或多次-->
<!ELEMENT student (name,age,sex)>
<!ELEMENT name (#PCDATA)>
<!ELEMENT age (#PCDATA)>
<!ELEMENT sex (#PCDATA)>
<!--xml必须要按照这个顺序写，否则会报错-->
<!ATTLIST student number ID #REQUIRED>
<!--#REQUIRED表示这个number是必须含有的-->
```
xml文档
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE students SYSTEM "student.dtd">

<students>
	<student number="s1">
		<name>tom</name>
		<age>18</age>
		<sex>male</sex>
	</student>
	<student number="s2">
		<name>tom</name>
		<age>18</age>
		<sex>male</sex>
	</student>
</students>
```

#### Schema

dtd虽然简单，但是有着诸多问题，比如它并不能规定文本的内容

Schema解决了这个问题

后缀名 `.xsd`


xsd文档**读懂注释处即可**
```xml
<?xml version="1.0"?>
<xsd:schema xmlns="http://www.itcast.cn/xml"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        <!--这里是定义了一个别名xsd来代表后面的一串地址-->
        <!--如果xmlns后面是空的话则代表没有前缀别名-->
        targetNamespace="http://www.itcast.cn/xml" elementFormDefault="qualified">
    <xsd:element name="students" type="studentsType"/>
    <xsd:complexType name="studentsType">
        <xsd:sequence>
            <xsd:element name="student" type="studentType" minOccurs="0" maxOccurs="unbounded"/>
        </xsd:sequence>
    </xsd:complexType>
    <xsd:complexType name="studentType">
        <xsd:sequence>
            <xsd:element name="name" type="xsd:string"/>
            <xsd:element name="age" type="ageType" />
            <xsd:element name="sex" type="sexType" />
            <!--规定了书写的顺序-->
        </xsd:sequence>
        <xsd:attribute name="number" type="numberType" use="required"/>
    </xsd:complexType>
    <xsd:simpleType name="sexType">
        <xsd:restriction base="xsd:string">
            <!--规定了sex的类型-->
            <xsd:enumeration value="male"/>
            <xsd:enumeration value="female"/>
            <!--规定了sex的种类，只能有两种-->
        </xsd:restriction>
    </xsd:simpleType>
    <xsd:simpleType name="ageType">
        <xsd:restriction base="xsd:integer">
            <!--规定了age的类型-->
            <xsd:minInclusive value="0"/>
            <xsd:maxInclusive value="256"/>
            <!--规定了age的最大最小值-->
        </xsd:restriction>
    </xsd:simpleType>
    <xsd:simpleType name="numberType">
        <xsd:restriction base="xsd:string">
            <xsd:pattern value="heima_\d{4}"/>
            <!--规定必须是heima_开头后跟四位数字-->
        </xsd:restriction>
    </xsd:simpleType>
</xsd:schema> 

```
xml文档
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- 
	1.填写xml文档的根元素
	2.引入xsi前缀.  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	3.引入xsd文件命名空间.  xsi:schemaLocation="http://www.itcast.cn/xml  student.xsd"
	4.为每一个xsd约束声明一个前缀,作为标识  xmlns="http://www.itcast.cn/xml"
 -->
<students xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns="http://www.itcast.cn/xml"
          xsi:schemaLocation="http://www.itcast.cn/xml  student.xsd"
>
    <student number="heima_0001">
        <name>tom</name>
        <age>18</age>
        <sex>male</sex>
    </student>

</students>
```

### 解析

操作xml文档，将文档的数据读取到内存中

操作XML文档
- 解析（读取）：将文档中的数据读取到内存中
- 写入：将内存中的数据保存到XML中，持久化的存储

---
+ 解析XML的方式
1. DOM：将标记语言一次性载入内存中，在内存中形成一个DMO树
    - 优点：操作方便，可以对文档进行CRUD的所有操作
    - 缺点：消耗内存
    - 服务器端

2. SAX：逐行读取，基于事件驱动
    - 优点：占内存十分小
    - 缺点：只能读取，不能增删改
    - 移动端使用

+ XML常见的解析器
    1. JAXP：sun公司提供的解析器，支持DOM和SAX两种思想，性能差，了解
    2. DOM4J：一款优秀的解析器
    3. Jsoup：可以使用类似于JQuery的方式来解析
    4. Pull：Android操作系统内置的解析器，sax方式

#### Jsoup快速入门

快速入门：

步骤
1. 导入jar包
2. 获取Document对象
3. 获取对应的标签

```java
//2. 获取Document对象，根据xml文档来获取
//2.1 获取xml的路径
String path = JsoupDemo1.class.getClassLoader().getResource("xml/student.xml").getPath();
//2.2 解析xml文档，加载文档进内存，获取DOM树
Document document = Jsoup.parse(new File(path), "utf-8");
//返回一个Elements的集合（ArrayList）
Elements elements = document.getElementsByTag("elements");
System.out.println(elements.size());
//3.1 获取第一个elements的element对象
Element element = elements.get(0);
String text = element.text();
System.out.println(text);
```
---
- Jsoup是一个工具类，可以解析html文档或者xml文档，返回Document对象
    - parse方法：解析html或xml文档，返回Document
        - `parse(File in,Stirng charsetName)`:解析xml或html文件
        - `parse(String html)`解析html字符串
        - `parse(URL url,int timeoutMillis)`通过网络来获取html文件

---
- Document：文档对象，代表内存中的Dom树
    - 获取Element对象
        - `getElementsByTag(String tagName)`:根据标签名称获取元素对象集合
        - `getElementsByAttribute(String key)`根据属性名称获取元素对象集合
        - `getElementsByAttributeValue(String key,String value)`根据对应的属性名和属性值获取元素对象的集合
        - `getElementsById(String id)`通过id属性值获取唯一的element对象
---
- Elements，元素Elements对象的集合，可以当做ArrayList集合来使用
---
- Element元素对象
        - 也可以获取Element对象
         - `getElementsByTag(String tagName)`:根据标签名称获取元素对象集合
        - `getElementsByAttribute(String key)`根据属性名称获取元素对象集合
        - `getElementsByAttributeValue(String key,String value)`根据对应的属性名和属性值获取元素对象的集合
        - `getElementsById(String id)`通过id属性值获取唯一的element对象
    - 获取属性值
        - `String attr(String key)`根据属性名称获取属性值
    - 获取文本内容
        - `String text()`获取所有标签的纯文本内容
        - `String html()`获取标签体的所有内容（包括子标签的标签和文本内容）
---
- Node节点对象
    - 是Document和Element元素的父类

 
#### Jsoup选择器查询

快捷查询方式：

- selector：选择器的方式

使用的方法：`Elements select(String sccQuery)`
    
语法是css的选择器语法，参考文档中selector类

---

- XPath : XML路径语言，确定XML文档中某部分位置的

语法：[参考](https://www.w3school.com.cn/xpath/xpath_syntax.asp)

需要额外导入一个jar包

```java
public static void main(String[] args) throws IOException, XpathSyntaxErrorException {
    URL url = new URL("http://baidu.com/");
    Document document = Jsoup.parse(url, 10000);
    //创建xpath的document
    JXDocument jxDocument = new JXDocument(document);
    //结合语法来查询
    List<JXNode> jxNodes = jxDocument.selN("//span/div/span");
    //返回一个JSNode
    for (JXNode jxNode : jxNodes) {
        System.out.println(jxNode);
    }

}
```