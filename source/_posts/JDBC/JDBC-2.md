---
title: JDBC-2
date: 2019-09-02 11:11:40
tags: 
- JDBC
categories: 
- 后台
- JDBC
---

<center>
引言：

数据库连接池

JDBC Template
</center>

<!--more-->

--------

# 数据库连接池

**为什么要使用数据库连接池？**

原本的逻辑是，用户访问数据库要先向操作系统请求连接，访问完数据库把连接删除

这样的效率十分低下

数据库连接池就可以解决这个问题，它把所有的连接放在一个容器当中，当用户访问完成时就会归还连接到这个容器中，效率更高

## 概念

一个存放数据库连接的容器

当系统初始化好后，容器被创建，容器中会申请一些连接对象，

当用户来访问数据库时，从容器中获取连接对象，

用户访问完成后，会将连接对象归还给容器

好处：
1. 节约资源
2. 高效

## DataSource

**DataSource接口由驱动程序供应商实现。** 

### 包路径
```
javax.sql
```
### 方法
1. 获取连接
```java
getConnection()
```
2. 归还连接
```java
close()
//如果连接对象是连接池提供的，那么close方法将不再关闭连接，而是归还连接
```

# 两个数据库连接池

## C3P0(旧的技术)
基本使用：
1. 导入两个jar包(JDBC的jar包也要导入)
    - `c3p0-0.9.5.2`
    - `mchange-commons-java-0.2.12`
2. 定义配置文件
    - 配置文件通常在应用程序类路径顶层的标准名称`c3p0.properties或c3p0-config.xml`下查找
    - 路径：直接将文件放在src目录下即可
3. 创建核心对象 数据库连接池对象 ComboPooleDataSource
4. 获取连接：getConnection
```java
    public static void main(String[] args) throws SQLException {
        //1. 创建数据库连接对象
        DataSource ds= new ComboPooledDataSource();
        //2. 获取连接对象
        Connection conn = ds.getConnection();
        //3. 打印
        System.out.println(conn);
    }
```
c3p0-config.xml配置文件
```xml
<c3p0-config>
  <!-- 使用默认的配置读取连接池对象 -->
  <default-config>
  	<!--  连接参数 -->
    <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/database?serverTimezone=UTC</property>
    <property name="user">root</property>
    <property name="password">1046qwer.</property>
    
    <!-- 连接池参数 -->
    <property name="initialPoolSize">5</property>
    <property name="maxPoolSize">10</property>
    <property name="checkoutTimeout">3000</property>
  </default-config>

  <named-config name="otherc3p0"> 
    <!--  连接参数 -->
    <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/database?serverTimezone=UTC</property>
    <property name="user">root</property>
    <property name="password">1046qwer.</property>
    
    <!-- 连接池参数 -->
    <property name="initialPoolSize">5</property>
    <property name="maxPoolSize">8</property>
    <property name="checkoutTimeout">1000</property>
  </named-config>
</c3p0-config>
```


## Druid(阿里巴巴提供)

步骤：

1. 导入jar包
2. 定义配置文件
    - 是properties形式的
    - 可以称任意名称，可以放在任意目录下
3. 加载配置文件
4. 获取数据库连接池对象：通过一个工厂类来获取`DruidDataSourceFactory`
5. 获取连接`getConnection`
示例代码
```java
public class druid {
    public static void main(String[] args) throws Exception {
        //1. 导入jar包
        //2. 定义配置文件
        //3. 加载配置文件
        Properties pro = new Properties();
        InputStream is = druid.class.getClassLoader().getResourceAsStream("druid.properties");
        pro.load(is);
        //4. 获取连接对象
        DataSource ds = DruidDataSourceFactory.createDataSource(pro);
        //5. 获取连接
        Connection conn = ds.getConnection();
        System.out.println(conn);
    }
}

```
### 定义工具类
准备一个JDBCUtils类

内部要有
1. 静态代码块加载配置文件
2. 获取连接方法
3. 释放资源
4. 获取连接池的方法
```java
/**
 * @author BlackKnight
 * 工具类
 */
public class JdbcUtils {
    private static DataSource ds;

    static {
        //1. 加载配置文件
        try {
            Properties pro = new Properties();
            pro.load(JdbcUtils.class.getClassLoader().getResourceAsStream("druid.properties"));
            ds = DruidDataSourceFactory.createDataSource(pro);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取连接
     *
     * @return
     * @throws SQLException
     */
    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }

    /**
     * 释放资源
     * @param stmt
     * @param conn
     */
    public static void close(Statement stmt, Connection conn) {
        if (stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

    }

    public static void close(ResultSet rs, Statement stmt, Connection conn) {
        close(null,stmt,conn);
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    public static DataSource getDataSource(){
        return ds;
    }

}
```
使用工具类
```java
public class druid {
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement preStat = null;
        try {
            conn = JdbcUtils.getConnection();
            String sql = "insert into deliver values(1,?,?)";
            preStat = conn.prepareStatement(sql);
            preStat.setString(1,"李白");
            preStat.setString(2,"3000");
            int i = preStat.executeUpdate();
            System.out.println(i);
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JdbcUtils.close(preStat,conn);
        }
    }
}
```

# Spring JDBC

Spring框架提供的对JDBC简单封装

提供了一个JDBCTemplate对象简化JDBC的开发

步骤：
1. 导入jar包
2. 创建JDBCTemplate对象，依赖于数据源DataSource
    `JdbcTemplate template = new JdbcTemplate(ds)`
3. 使用JDBCTemplate方法完成CRUD操作
    - `update()`执行DML语句
    - `queryForMap()`查询结果，将结果封装为Map集合
    - `queryForList()`查询结果将结果封装为List集合
    - `query()`查询结果，将结果封装为JavaBean对象
    - `queryForObject`查询结果，将结果封装为对象

示例代码
```java
public class JdbcTemplateDemo {
    public static void main(String[] args) {
        //1. 导入jar包
        //2. 创建JDBCTemplate对象
        JdbcTemplate template = new JdbcTemplate(JdbcUtils.getDataSource());
        //3. 调用方法
        String sql = "update deliver set bank = 5000 where id = ?";
        int count = template.update(sql, 3);
        System.out.println(count);
    }
}

```