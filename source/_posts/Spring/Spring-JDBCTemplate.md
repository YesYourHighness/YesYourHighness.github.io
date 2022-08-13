---
title: Spring_JDBCTemplate
date: 2020-03-13 23:54:03
tags:
- Spring
categories:
- Spring
---


<center>
引言：

Spring——JDBCTemplate    
</center>

<!--more-->
---

# JDBC Template

Spring为了简化JDBC的持久化操作，提供了`JDBC Template`

```
//原本操作数据库的流程
我们的代码 -> JDBC API -> JDBC驱动 -> MySql

//现在
我们的代码 -> JDBC Template -> JDBC API -> JDBC驱动 -> MySql
```
Maven配置：
```xml
<!--Mysql-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.44</version>
</dependency>
<!--JDBC-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
<!--事务管理-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
```

配置`SpringConfig`文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--约束文件-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!--配置JDBC-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource" >
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/selection_course?useUnicode=true&amp;characterEncoding=utf-8" />
        <property name="username" value="root" />
        <property name="password" value="root" />
    </bean>
    <!--配置JDBCTemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
</beans>
```

## 增删改相关方法
### update方法（对数据进行增删改）
```java
int update(String sql, Obejct[] args)
//参数是SQL语句和一个数组
int update(String sql, Obejct... args)
//参数是SQL和任意个参数
```
例如以下的操作
```java
String sql1 = "insert into student(name,sex) values(?,?)";
String sql2 = "update student set sex=? where id=?";
jdbcTemplate.update(sql1,new Object[]{"李白","男"});
jdbcTemplate.update(sql2,"女",1);
```

### batchUpadate方法（批量增删改操作）
```java
int[] batchUpdate(String[] sql)
//执行一个数组内的全部sql
int[] batchUpdate(String sql,List<Obejct[]> args)
//可以将一个sql语句分别插入不同的值执行
```
例如以下操作
```java
@Test
public void testBatchUpdate1(){
    String[] sql = {
            "insert into student(name,sex) values('杜哺','男')",
            "insert into student(name,sex) values('孟浩然','男')",
            "insert into student(name,sex) values('李清照','女')"};
    jdbcTemplate.batchUpdate(sql);
}
@Test
public void testBatchUpdate2(){
    String sql = "insert into selection(student,course) values(?,?)";
    List<Object[]> list = new ArrayList<>();
    list.add(new Object[]{13,1001});
    list.add(new Object[]{15,1002});
    list.add(new Object[]{14,1003});
    jdbcTemplate.batchUpdate(sql,list);
}
```

## 查询相关方法

### 简单的查询
查询结果只有一个字段，例如查询个数，查询所有的男生名字等等
- 获取一个
```java
T queryForObeject(String sql, Class<T> type)
//参数是Sql语句和查询结果是一个什么类
T queryForObeject(String sql, Object[] args,Class<T> type)
T queryForObeject(String sql, Class<T> type, Object... arg)
```
例如以下操作
```java
String sql = "select count(*) from student";
int count = jdbcTemplate.queryForObject(sql, Integer.class);
System.out.println(count);
```

----

- 获取多个

```java
List<T> queryForList(String sql, Class<T> type)
List<T> queryForList(String sql, Obejct[] args,Class<T> type)
List<T> queryForList(String sql, Class<T> type, Object... arg)
```
例如下面的操作
```java
String sql = "select name from student where sex=?";
List<String> names = jdbcTemplate.queryForList(sql, String.class,"女");
System.out.println(names);
```

### 复杂的查询
复杂的查询会封装为`Map`类型

- 获取一个
```java
Map queryForMap(String sql)
Map queryForMap(String sql, Object[] args)
Map queryForMap(String sql, Object... arg)
```
例如下面的操作
```java
String sql = "select * from student where id=?";
Map<String, Object> stu = jdbcTemplate.queryForMap(sql, 13);
System.out.println(stu);
```


- 获取多个

```java
List<Map<String, Object>> queryForList(String sql)
List<Map<String, Object>> queryForList(String sql, Object[] args)
List<Map<String, Object>> queryForList(String sql, Object... arg)
```
看完会发现，只要我们不指定返回的类型，他就是一个查询多个字段获取多个字段的方法

例如下面操作
```java
String sql = "select * from student";
List<Map<String, Object>> stus = jdbcTemplate.queryForList(sql);
System.out.println(stus);
```
### 查询复杂对象（并封装为实体对象）



方法主要有：
- 获取一个
```java
T queryForObject(String sql, RowMapper<T> mapper)
T queryForObject(String sql, Obeject[] args,RowMapper<T> mapper)
T queryForObject(String sql, RowMapper<T> mapper, Object... arg)
```
例如下面的操作，其中`Student`类已经写出，只是一个示意
```java
String sql = "select * from student where id = ?";
Student student = jdbcTemplate.queryForObject(sql, new RowMapper<Student>() {
    @Override
    public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
        Student stu = new Student();
        stu.setId(rs.getInt("id"));
        stu.setName(rs.getString("name"));
        stu.setSex(rs.getString("sex"));
        stu.setBorn(rs.getDate("born"));
        return stu;
    }
}, 13);
System.out.println(student);
```

- 获取多个
```java
List<T> query(String sql, RowMapper<T> mapper)
List<T> query(String sql, Object[] args,RowMapper<T> mapper)
List query(String sql, RowMapper<T> maooer,Object... arg)
```
**`RowMapper`接口可以实现封装为实体对象**,例如下面这个操作
```java
String sql = "select * from student";
List<Student> student = jdbcTemplate.query(sql, new RowMapper<Student>() {
    @Override
    public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
        Student stu = new Student();
        stu.setId(rs.getInt("id"));
        stu.setName(rs.getString("name"));
        stu.setSex(rs.getString("sex"));
        stu.setBorn(rs.getDate("born"));
        return stu;
    }
});
System.out.println(student);
```

