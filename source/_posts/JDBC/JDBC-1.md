---
title: JDBC-1
date: 2019-09-01 21:56:33
tags: 
- JDBC
categories: 
- 后台
- JDBC
---


<center>
引言：

把Java和数据库联系起来

这就是JDBC

</center>

<!--more-->

------

# JDBC

概念：Java DataBase Connectivity Java数据库连接

本质：官方定义的一套操作所有关系型数据库的规则，即接口。

各个数据库厂商去实现这套接口，提供数据库驱动jar包。我们可以使用这套接口编程，真正执行的代码是驱动jar包中的实现类

## 快速入门

步骤：


示例代码：
```java
//1. 导入驱动jar包
//   - 复制jar包到目录下
//   - 右键添加到library`add as library`
//2. 注册驱动
/*现在大部分的jar包都可以直接的注册，只要jar包内含有 META-INF/services/java.sql.Dirver 这个文件，这个文件内已经有 com.mysql.cj.jdbc.Driver 这条信息， 所以可以自动的注册驱动，如果不能自动注册驱动，再来手动的注册驱动
*/
Class.forName("com.mysql.cj.jdbc.Driver");
//System.setProperty("jdbc.drivers","com.mysql.cj.jdbc.Driver");//也可以使用这条语句进行注册，这种注册方式可以通过":"分离来一次填入多个驱动
/* 执行这条语句后,我们就有了一个驱动器对象 DriverManager*/

//3. 获取数据库连接对象
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/database?serverTimezone=UTC","root","root");
/*sql语句中使用了类似URL的相似语法来描述数据源
 在这里我们能看出这个语句的格式是：
jdbc:[数据库运营商]://[要连接的数据库的URL]:[端口号]/[建立的数据库]?serverTimezone=UTC
*/
//4. 定义SQL语句
String sql = "update deliver set bank = 500 where id = 1";
//5. 通过获得的连接获取执行sql的对象 Statement
Statement statement = conn.createStatement();
//6. 执行SQL
int count = statement.executeUpdate(sql);
/*executeUpdate(String sql)方法可以执行sql语句，并且会返回一个int类型的值，这个值表示此sql语句影响的行数*/
//7. 处理结果   
System.out.println(count);
//8. 释放资源
statement.close();
conn.close();
```

# 详解JDBC各个对象

概览：
1. DirverManager:驱动管理对象
2. Connection：数据库连接对象
3. Statement：执行SQL对象
4. ResultSet：结果集对象
5. PreparedStatement：执行SQL对象（是Statement的子接口，但功能更加强大）

## DirverManager

DirverManager是驱动管理类

功能：
1. 注册驱动
2. 获取数据库连接
### 包路径
```java
java.sql
```
### 注册驱动
现在大部分的jar包都可以直接的注册，只要jar包内含有 `META-INF/services/java.sql.Dirver` 这个文件，这个文件内已经有 `com.mysql.cj.jdbc.Driver` 这条信息， 所以可以自动的注册驱动，如果不能自动注册驱动，再来手动的注册驱动

mysql5.0 版本后注册驱动的代码可以省略，jar包会自动注册
```java
static void registerDriver(Driver driver) 
//注册与给定的驱动程序 DriverManager 。 

//写代码使用:
Class.forName("com.mysql.cj.jdbc.Driver");
//Class.forName用来注册类
//此类中含有static静态代码块,我们无需自己调用registerDriver方法
/*
static {
    try {
        DriverManager.registerDriver(new Driver());
        //这里帮我们执行了此方法
    } catch (SQLException var1) {
        throw new RuntimeException("Can't register driver!");
    }
}
*/    
```


### 获取数据库连接
sql语句中使用了类似URL的相似语法来描述数据源,
url的MySQL语法：`jdbc:mysql://ip地址(域名):端口号/数据库名称`
```java
static Connection getConnection(String url, String user, String password) 
//尝试建立与给定数据库URL的连接。  
//静态方法，可以用类名直接调用
//url：指定的路径
//url的MySQL语法：jdbc:mysql://ip地址(域名):端口号/数据库名称
```
如果连接的是本机`127.0.0.1`的mysql服务器，并且myql服务端默认为3306，则url可以简化为
```java
"jdbc:mysql:///数据库 名称"
// 这里有三个/
```
## Connection
Connection数据库连接对象

功能：
1. 获取执行SQL的对象statement
2. 管理事务

### 包路径
```java
java.sql
```
### 获取执行sql的对象
```java
Statement createStatement() 
//创建一个Statement对象，用于将SQL语句发送到数据库。

PreparedStatement prepareStatement(String sql) 
//创建一个 PreparedStatement对象，用于将参数化的SQL语句发送到数据库。  
```
### 管理事务
```java
# 开启事务
void setAutoCommit(boolean autoCommit) 
//将此连接的自动提交模式设置为给定状态。
//设置值为false，即开启事务
# 提交事务
void commit() 
//使自上次提交/回滚以来所做的所有更改都将永久性，并释放此 Connection对象当前持有的任何数据库锁。  
# 回滚事务
void rollback() 
//撤消在当前事务中所做的所有更改，并释放此 Connection对象当前持有的任何数据库锁。  
```

## Statement
用于**执行静态SQL语句**并返回其生成的结果的对象

功能：
1. 执行sql

### 包路径
```java
java.lang
```

### 执行sql

```java
boolean execute(String sql) 
/*执行任意的SQL语句，这可能会返回多个结果集或者是影响个数 ,通常用于 由用户提供的交互式查询
如果返回值是true，说明第一个执行结果是一个结果集，此时可以调用getResultSet()方法来获得结果集
反之，返回false，此时可以调用getUpdateCount获得影响个数

*/
int executeUpdate(String sql) 
//执行DML(INSERT,UPDATE,或 DELETE语句)语句,执行DDL语句(create,alter,drop)语句，不能执行查询SELECT语句，返回值是这条sql语句影响的行数
//可以通过返回值来判断语句是否成功,大于0则成功

ResultSet executeQuery(String sql) 
//执行查询的SQL语句，该语句返回单个ResultSet对象。 返回值是一个结果集对象
```
### ResultSet
```java
java.sql
```

数据库的列表就是一个结果集，一开始结果集的光标在最上方，表头的部分，我们需要先将它**移动至下一行**才能读取数据`next()`

获取数据`getXxx(参数)`Xxx代表数据类型，参数可以写int型的列编号 **从1开始**，可以写String型的字段

例如：这样的一张表
```
+----+------+----------+
| id | name | PASSWORD |
+----+------+----------+
|  1 | 李   | 123      |
|  2 | 王   | 456      |
|  3 | 张   | 789      |
+----+------+----------+
```
```java
while (resultSet.next()){
    System.out.print(
        resultSet.getInt(1)
        + "-"
        + resultSet.getString(2)
        + "-"
        + resultSet.getInt(3)+"\r\n"
    );
}
/*
注意：
1. 在sql表中，第一列对应的就是1，不是0
2. 在使用getString()这个方法时，参数可以是列数，也可以是列表的表头名称，例如，可以如下这么写
3. 当get方法类型和获取的参数类型不一致的时候，每个get方法都会进行合理的数值转换
*/
while (resultSet.next()){
    System.out.print(
        resultSet.getInt("id")
        +"-"+
        resultSet.getString("name")
        +"-"+
        resultSet.getInt("password")+"\r\n"
    );
}
/*打印出结果
1-李-123
2-王-456
3-张-789
*/
```
#### ResultSet结果集方法
```java
boolean next();
//将结果集向前移动一行，如果已经是最后一行的后面，则返回false，注意：初始情况下必须先调用该方法才能移动到第一行

Xxx getXxx()
// 各种类型的get方法

void close();
//立即关闭当前结果集
```

### 管理连接、语句、结果集
每个Connection对象都可以创建一个或者多个Statement语句，同一个Statement对象可以用于多个不相关的命令和查询，**但是**一个Statement对象最多只能有一个打开的结果集

这就是说，如果我们要进行多组插入或者删除，只需要一个Statement对象即可

但是如果需要执行多个查询操作，并且要同时分析查询结果，我们要创建多个Statement对象

**但实际上，我们不需要同时处理多个结果集，对数据库进行组合查询比使用java遍历结果集要高效的多**

## PreparedStatement
> prepared statement 预备语句：一个带有宿主变量的查询语句，可以进行动态的查找

优点：
1. 安全，防止代码注入问题
2. 高效，改进了查询的性能


### 通过连接获得预备语句
```java
PreparedStatement Connection.prepareStatement(String sql)
```
构造完成后，我们还不能直接的执行sql语句，我们必须先进行一下绑定
### 与变量?绑定
```java
setXxx(参数1，参数2)
/* 
    参数一：?的位置编号，从1开始
    参数二: ?的值
*/
```
**注意： 只有查询涉及变量时，才应该使用预备语句**

如果想要重用已经执行过的sql语句，必须先使用`setXxx()`或者`clearParameters`方法重新设置宿主变量和绑定，否则这些绑定关系都不会改变

### 示例
原本
```java
String sql = "select * from  user where name = ? and password = ?";
//这样会导致sql注入问题，我输入 (a' or 'a' = 'a) 所有人都可以进入
```
现在
```java
String sql = "select * from  user where name = ? and password = ?";
preStat = conn.prepareStatement(sql);
//设置?的值
preStat.setString(1,name);
preStat.setString(2,pass);
//这里将不需要传参
rs = preStat.executeQuery();
```
### 预备语句方法
```java
void serXxx(int n, Xxx x)
//设置第n个?的值为x

void clearParameters()
//清除预备语句中所有的当前参数

ResultSet executeQuery()
//执行查询语句，并返回一个结果集对象

int executeUpdate()
// 执行INSERT、UPDATE、DELETE预备sql语句
```


# demo示例
## 1. 简单的demo
```java
public class JdbcDemo2 {
    public static void main(String[] args) {
        Statement stat = null;
        Connection conn =null;
        try {
            //1. 注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            //2. 定义sql
            String sql = "insert into deliver values(null,'李',3000)";
            //3. 获取Connection对象
            conn = DriverManager.getConnection("jdbc:mysql:///database?serverTimezone=UTC", "root", "root");
            //4. 执行sql语句
            stat = conn.createStatement();
            int i = stat.executeUpdate(sql);
            if(i>0){
                System.out.println("添加成功");
            }else {
                System.out.println("添加失败");
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            //避免空指针异常
            if(stat!=null){
                try {
                    stat.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(conn!=null){
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}

```


## 2. 完整的demo
创建了Jdbcutils的工具类`JdbcUtils`
```java
/**
 * JDBC工具类
 */
public class JdbcUtils {
/*文件只需要读取一次即可,可以使用静态代码块
静态代码块会随着类的加载而加载，且只加载一次
这里定义一些静态的变量，因为静态代码块只能访问静态变量*/
private static String url;
private static String user;
private static String password;
private static String driver;

static {
    try {
        //获取资源配置文件
        //1. 创建Properties
        Properties pro = new Properties();
        //2. 加载文件
        //src下文件的获取方式-->ClassLoader类加载器
        ClassLoader cl = JdbcUtils.class.getClassLoader();
        URL res = cl.getResource("jdbc.properties");
        //返回一个URL对象，使用getPath方法可以定位一个字符串路径
        String path = res.getPath();
        //路径是一个绝对路径
        pro.load(new FileReader(path));
        //3. 获取数据，赋值
        url = pro.getProperty("url");
        user = pro.getProperty("user");
        password = pro.getProperty("password");
        driver = pro.getProperty("driver");
        //4. 注册驱动
        Class.forName(driver);
    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
    }

}

//静态方法，直接可以使用类名调用

/**
    * 获取连接
    *
    * @return 连接对象
    */
public static Connection getConnection() throws SQLException {
    return DriverManager.getConnection(url,user,password);
}

/**
    * 释放资源
    *
    * @param stat
    * @param conn
    */
public static void close(Statement stat, Connection conn) {
    if (stat != null) {
        try {
            stat.close();
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

/**
    * 重载函数
    * 释放资源
    *
    * @param rs
    * @param stat
    * @param conn
    */
public static void close(ResultSet rs, Statement stat, Connection conn) {
    if (stat != null) {
        try {
            stat.close();
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
    if (rs != null) {
        try {
            rs.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

}//end

```
配置文件`jdbc.properties`
```
url = jdbc:mysql:///database?serverTimezone=UTC
user = root
password = root
driver = com.mysql.cj.jdbc.Driver
```

保存的类对象`Person`
```java
public class JdbcDemo4 {
    public static void main(String[] args) {
        List<Person> list = new JdbcDemo4().findAll();
        System.out.println(list);
    }

    public List<Person> findAll() {
        Connection conn = null;
        Statement stat = null;
        ResultSet rs = null;
        List<Person> list = null;
        try {
            //使用工具类
            conn = JdbcUtils.getConnection();
            String sql = "select * from  deliver";
            stat = conn.createStatement();
            rs = stat.executeQuery(sql);
            Person p = null;
            list = new ArrayList<Person>();
            while (rs.next()){
                int id = rs.getInt(1);
                String name = rs.getString(2);
                int bank = rs.getInt(3);
                p = new Person();
                p.setId(id);
                p.setName(name);
                p.setBank(bank);
                list.add(p);
            }
        }catch (SQLException e) {
            e.printStackTrace();
        } finally {
            //使用工具类
            JdbcUtils.close(rs,stat,conn);
        }
        return list;
    }
}
```

## 3. 登录的demo
```java
public class JdbcDemo5 {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String name = sc.next();
        String password = sc.next();
        if (new JdbcDemo5().login(name, password)) {
            System.out.println("登陆成功");
        } else {
            System.out.println("登录失败");
        }
    }

    public boolean login(String name, String pass) {
        Connection conn = null;
        PreparedStatement preStat = null;
        ResultSet rs = null;
        if (name != null && pass != null) {
            try {
                conn = JdbcUtils.getConnection();
                String sql = "select * from  user where name = ? and password = ?";
                preStat = conn.prepareStatement(sql);
                preStat.setString(1,name);
                preStat.setString(2,pass);
                rs = preStat.executeQuery();
                return rs.next();
            } catch (SQLException e) {
                e.printStackTrace();
            } finally {
                JdbcUtils.close(rs, preStat, conn);
            }
            return false;
        } else {
            return false;
        }
    }
}
```

## 支付操作的demo
```java
public class JdbcDemo6 {
public static void main(String[] args) {
    Connection conn = null;
    PreparedStatement preStat1 = null;
    PreparedStatement preStat2 = null;
    ResultSet rs = null;
    try {
        conn = JdbcUtils.getConnection();
        //开启事务
        conn.setAutoCommit(false);
        String sql1 = "update deliver set bank = bank -? where id = ?";
        String sql2 = "update deliver set bank = bank +? where id = ?";
        preStat1 = conn.prepareStatement(sql1);
        preStat2 = conn.prepareStatement(sql2);
        preStat1.setString(1,"500");
        preStat1.setString(2,"3");
        preStat2.setString(1,"500");
        preStat2.setString(2,"2");
        preStat1.executeUpdate();
        preStat2.executeUpdate();
        //提交
        conn.commit();
    } catch (Exception e) {
        e.printStackTrace();
        if(conn!=null){
            try {
                //回滚
                conn.rollback();
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
        }
    }finally {
        JdbcUtils.close(rs,preStat1,conn);
        JdbcUtils.close(null,preStat2,null);
    }

    }
}
```
