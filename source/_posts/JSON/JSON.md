---
title: JSON
date: 2019-10-31 19:59:20
tags: 
- JSON
categories: 
- 后台
- JSON
---

<center>
引言：

JSON：Js对象表示法
</center>

<!--more-->

# JSON
> JavaScript Object Notation  ： Js对象表示法

一种存储、交换信息的语法

## 语法

1. 基本规则
```
* 数据在名称/值对中：json数据是由键值对构成的
    * 键用引号（单双引号都可以）勾起来，也可以不使用引号
    * 值的取值类型
        1. 数字（可以直接写）
        2. 字符串（双引号中）
        3. 布尔值
        4. 数组（方括号中）
        5. 对象（花括号中）
        6. null
* 数据由逗号分隔：多个键值对由逗号分隔
* 花括号保存对象：使用{}定义json格式
* 方括号保存数组：[]
```
```js
//1. 定义基本格式
var person = {"name": "张三", "age": 23, "gender": true};
//2. 嵌套格式
//{}嵌套[]
var persons = {
    "persons": [
        {"name": "张三", "age": 23, "gender": true},
        {"name": "李四", "age": 23, "gender": true},
        {"name": "王五", "age": 23, "gender": true}
    ]
};
//[]嵌套{}
var ps = [
    {"name": "张三", "age": 23, "gender": true},
    {"name": "李四", "age": 23, "gender": true}
];
```
2. 获取数据
    1. `json对象.键名`键名不需要加引号
    2. `json对象["键名"]`这时需要加引号
    3. `数组对象[索引]`
3. 遍历数据
```js
var person = {"name": "张三", "age": 23, "gender": true};
for(var key in person){
    // alert(key+":"+person.key);
    // key默认为一个字符串，不能用.来调用它
    alert(key+":"+person[key]);
}


var ps = [
{"name": "张三", "age": 23, "gender": true},
{"name": "李四", "age": 23, "gender": true}
];
for(var i=0;i<ps.length;i++){
    for (var pKey in ps[i]) {
        alert(pKey+":"+ps[i][pKey])
    }
}
```

## JSON数据与JAVA对象的相互转换

JSON解析器：一些封装好的工具类直接可以帮我们解析
 - 常见的解析器：Jsonlib，Gson，fastjson，jackson

### JSON转为java对象
了解，使用的并不多

使用步骤：
1. 导入jackson相关jar包
2. 创建jackson核心对象，ObjectMapper
3. 调用ObejctMapper的相关方法进行转换

- 转换相关方法：
```
readValue(json,Obejct)
第一个参数为json字符串，第二个参数为对象
```
示例
```java
 String json = "{\"gender\":\"男\",\"name\":\"张三\",\"age\":23}";
ObjectMapper mapper = new ObjectMapper();
Person p = mapper.readValue(json, Person.class);
p.setName("李四");
System.out.println(p);
//Person{name='李四', age=23, gender='男'}
```




### Java对象转换JSON
使用步骤：
1. 导入jackson相关jar包
2. 创建jackson核心对象，ObjectMapper
3. 调用ObejctMapper的相关方法进行转换


- 转换相关方法：
```
writeValue(参数1,obj对象);
    参数1：
        File：将obj对象转换为Json字符串，保存到指定的文件中
        Writer：保存到字符输出流中
        OutputStream：保存到字节输出流中
writeValueAsString(obj):将对象专为json字符串
```

```java
//1.创建Person对象
Person p = new Person();
p.setName("张三");
p.setAge(20);
p.setGender("男");
//2. 创建Jackson的核心对象 ObjectMapper
ObjectMapper mapper = new ObjectMapper();
//3. 调用相关方法

String json = mapper.writeValueAsString(p); 
System.out.println(json);
//打印出{"name":"张三","age":20,"gender":"男"}
```
- 注解：
1. `@JsonIgnore`，排除属性
2. `@JsonFormat`，属性值的格式化

将注解加在JavaBean的成员变量上
```java
@JsonIgnore //忽略该属性
private Date birthday;

@JsonFormat(pattern = "yyyy-MM-dd")//格式化该属性
private Date birthday;
```

- 复杂集合的转换

List集合转换，会成为一个数组
```java
List<Person> ps = new ArrayList<>();
ps.add(p);
String s = mapper.writeValueAsString(ps);
System.out.println(s);
//[{"name":"张三","age":20,"gender":"男"}]
```

Map集合转换，和对象转换一样
```java
Map<String, Object> map = new HashMap<>();
map.put("name","张三");
map.put("age",23);
map.put("gender","男");
String s = mapper.writeValueAsString(map);
System.out.println(s);
//{"gender":"男","name":"张三","age":23}
```

## 注意
服务器响应的数据，在客户端使用时，要想当做json的数据格式使用：

以下方式二选一
1. `$.get(type)`将最后一个参数设置为"json"
2. 服务端设置MIME类型`response.setContentType("application/json;")`
