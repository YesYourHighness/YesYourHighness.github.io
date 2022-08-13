---
title: Mybatis复习
date: 2021-09-12 14:56:20
tags: 
- Mybatis
categories: 
- Mybatis

---

<center>
引言：Mybatis框架复习
</center>
<!--more-->

# Mybatis

## JDBC开发

JDBC开发我们连接数据库需要以下几步操作：

1. 编写Sql语句
2. 预编译
3. 设置参数
4. 执行Sql
5. 封装结果返回

> 为什么不使用JDBC，而要去使用框架呢？

1. 功能简单，Sql直接编写在代码中，高耦合（如果我们之后想更换其中的Sql语句还需要重新打包上线）

## ORM

> ORM：Object Relation Mapping 对象关系映射
>
> 将对象与数据库内的记录进行映射

实现了ORM的框架：

- Hibernate：一个**全自动ORM框架**
- Mybatis：一个**半自动ORM框架**

全自动：指一切过程（包括Sql的编写）都由框架来帮助我们实现

半自动：Sql由我们来写

> 为什么不用Hibernate，这么好用，Sql都不用自己写？

1. Sql由框架封装，没办法优化
2. 如果要自定义Sql，要去学习HQL，增加学习负担
3. Hibernate还是一个**全映射**框架，意味着Bean有多少字段，就会去数据库查多少字段，即使你只需要1个而已，也会全部查出（即一直回表）

## Mybatis工作原理

1. 读取**全局配置文件**（比如`mybatis-config.xml`）：配置了比如数据源、事务等相关逻辑
2. 加载**映射文件**（比如`Mapper1.xml Mapper2.xml`，需要在全局配置文件注册）
3. 构造**会话工厂**`SqlSessionFactory`：此对象可以创建SqlSession会话，
4. 构造会话对象`SqlSession`：此对象用来与数据库进行会话，注意这个对象不是线程安全的！
5. `Executor`执行器：会根据`SqlSession`传递的参数**动态的生成所需要执行的Sql语句**，**同时负责查询缓存的维护**
6. `MapperStatement`对象：`mapper`的一个sql对应一个`MapperStatement`对象，Sql的id就是`MapperStatement`的id
7. 输入参数映射
8. 输出参数映射

我们只需要使用**接口式编程**，**Mybatis会为我们创建代理对象，**代理对象去执行增删改查

注意：

1. 原生：`Dao` -> `DaoImpl`；mybatis: `Mapper` -> `xxMapper.xml`
2. `SqlSession`代表与数据库的一次会话，使用完成后必须关闭
3. `SqlSession`**不是线程安全**的！所以我们每次都要去获取新的`SqlSession`对象，而不是作为成员引用
4. `mapper`接口没有实现类，是因为`Mybatis`帮我们生成了一个代理对象！
5. 两个重要的配置文件：
   1. `Mybatis`全局配置文件（这个配置文件可以没有，可以通过new的方式获得载入`SqlSessionFactory`中）：配置数据源、事务管理器等
   2. Sql映射文件

## Mybatis全局配置文件

`<configuration>`是Mybatis框架全局配置文件的根元素，其子元素还有：

- `<properties>`：将配置外在化（比如配置在`properties`文件中）配置数据源，比如数据库驱动、数据库url、用户名、密码等

- `<settings>`：**配置Mybatis运行时的行为**

  | 设置参数                 | 作用                                                   | 有效值     | 默认值   |
  | ------------------------ | ------------------------------------------------------ | ---------- | -------- |
  | cacheEnabled             | 影响所有映射器中配置的缓存全局开关（配置的是二级缓存） | true/false | false    |
  | lazyLoadingEnabled       | 延迟加载的全局开关                                     | true/false | true     |
  | useColumnLabel           | 使用列标签代替列名                                     | true/false | true     |
  | defaultStatementTimeOut  | 设置超时时间，决定驱动等待数据库响应的秒数             | 正整数     | 没有设置 |
  | mapUnderscoreToCamelCase | 是否开启驼峰命名规则                                   | true/false | false    |

- `<typeAliases>`：**设置别名**，可以减少冗余的全限定名

  ```xml
  <typeAliases>
      <typeAlias alias="user" type="com.xx.xx.User">
          <!--配置后user就可以在mybatis配置文件的任何位置代替其全限定名--> 
  </typeAliases>
  ```

  - 如果不配置alias，默认就是类名的首字母小写后的名称

  - 注意**别名不区分大小写**

  - 也可以使用注解的方式`@Alias(value="user")`在类上加（会覆盖原来的别名）

  - 可以使用扫描包的形式，批量设置别名

    ```xml
    <typeAliases>
        <package name="com.xx.xx.pojo">
    </typeAliases>
    ```

- `<typeHandlers>`：**处理对象与数据库数据类型之间映射的桥梁**，之后详细说明，非常重要！

- `<objectFactory>`

- `<plugins>`：Mybatis支持**拦截调用**，拦截调用是通过插件来实现的

- `<environments>`：用于对环境进行配置，比如数据源、比如事务；可以用来配置多种环境，比如开发环境、生产环境、测试环境等

  - 子标签`<environment>`必须要配置`<transactionManager>`和`<dataSource>`
  - `<transactionManager type="">`其中**事务管理器**Mybatis有两种，但是一般Mybatis是与Spring一起使用的，所以直接使用Spring的事务管理即可
  - `<dataSource type="">`数据源有三种配置：UNPOOLED、POOLED、JNDI（所以当然使用POOLED这种方式）
  - 如果要使用自己的数据源，比如C3P0、Druid，需要**自定义数据源**：要实现`DataSourceFactory`接口

- `<databaseIdProvider>`：适应不同的数据库，然后再`select`语句中可以使用`databaseId`属性来标记这个sql去哪个数据库查

  ```xml
  <databaseIdProvider type="DB_VENDOR">
      <property name="MySQL" value="mysql" />
      <property name="Oracle" value="oracle" />
  </databaseIdProvider>
  ```

- `<mappers>`：配置Sql映射文件`mapper.xml`的位置；如果使用注解的方式，没有写`mapper`文件，也可以配置在这里（推荐使用xml文件的方式！解耦合）

  ```xml
  <mappers>
      <mapper resource="配置Mapper.xml文件的位置"/>
      <mapper class="配置到接口，接口上有@Select等注解">
      <package name="批量注册"/>
  </mappers>
  ```

## Mybatis映射文件

### 映射文件标签配置

Mybatis最重要的地方，Mybatis的强大就来源于映射文件的编写！

根为一个`<mapper>`标签，其子标签有很多：

- `<insert> <update> <delete>`：增删改，其子标签有

  - `id`：此Sql的唯一标识符，设置要与接口的方法名一致
  - `parameterType`：输入参数类型，可以**省略**
  - 关于返回值：mybatis为增删改自动封装了几个返回值
    - `integer / long`：操作的行数
    - `boolean`：操作了大于0行，返回true
  - 关于`insert`的主键自增：配置`useGeneratedKeys="true"`与`keyProperty="bean中的主键名"`即可

- `<select>`：查询操作

  - 要用`resultType`标识返回值的类型

  ```xml
  <mapper namespace="cn.yesmylord.login.demo.dao.UserDao">
      <select id="selectUserByUsername" resultType="cn.yesmylord.login.demo.pojo.User">
          select uid,username,password,salt from login_user where username = #{username};
      </select>
  </mapper>
  ```

  - 如果**返回值是一个集合**，`resultType`也要去**标识其中元素的类型**！

  - 如果想返回一个Map，key为列名，value为值，就使用`resultType="map"`

  - 如果想返回一个Map，key为表的键，value为封装后的记录，就用`resultType="封装的对象比如Student"`，然后使用`@MapKey("id")`注解让Mysql知道key为哪一个值

  - `resultMap`用来自定义映射关系！实现高级的结果集映射，对于不指定的会自动进行映射

    ```xml
    <resultMap>
        <id column="id" property="id"/>
        <!-- 用id专门来对应键 -->
        <result column="email" property="email"/>
        <!-- 其他普通的属性用result -->
    </resultMap>
    ```

- `<sql>`

- `<cache>`：配置缓存

- `<cache-ref>`

- `<resultMap>`

### 参数处理

> mybatis是如何通过`#{id}`就能获取到值的呢？

这里面涉及到参数处理过程

1、对于单个参数，mybatis不会做特殊处理，会直接返回结果

此时不管你`#{xxx}`内写什么，都可以返回正确的结果

2、对于多个参数，比如 `getUser(String username, String password)`

如果你使用`#{username} #{password}`是获取不到对应的值的！

> 原因：多参数时mybatis会做特殊处理，会将这些参数封装为一个map，map的`key`为`param1`、`param2`、`param3`，`value`为：传入的参数值

所以多参数需要使用`#{param1} #{param2}`获取，但是这样太反人类了，所以我们可以**使用注解**，像这样

````java
getUser(@Param("username")String username, @Param("password")String password)
````

这样使用`#{username} #{password}`就能获取到值了

> 原因：此时mybatis封装map时，就会用我们给的名作为key了！

3、如果参数很多，与POJO类一样，那么直接传POJO类就可以了

4、如果参数很多，但不是POJO类，我们可以传入map对象

5、如果参数很多，而且这个方法很常用，那我们需要编写一个TO（Transfer Object）

6、如果是一个集合，取法类似于`#{list[0]}`这样

### 源码探究

我们执行这样一个方法，打断点，进入深层查看：

```java
User libai = userDao.selectUserByUsername("李白");
```

第一层：MapperProxy代理类

```java
public class MapperProxy<T> implements InvocationHandler, Serializable {
    // 可以看到，这个MapperProxy是一个代理对象
    // 因为有这个代理对象，我们的接口才可以直接运行
	... 省略
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 如果是继承自Object的方法，直接跳过
        if (Object.class.equals(method.getDeclaringClass())) {
            try {
                return method.invoke(this, args);
            } catch (Throwable var5) {
                throw ExceptionUtil.unwrapThrowable(var5);
            }
        } else {
            MapperMethod mapperMethod = this.cachedMapperMethod(method);
            // 我们再进入下一层
            return mapperMethod.execute(this.sqlSession, args);
        }
    }

}
```

第二层：MapperMethod，虽然多，但是我们不需要管其他的，注意我们执行的是一个select方法就好

```java
public class MapperMethod {
    public Object execute(SqlSession sqlSession, Object[] args) {
        Object param;
        Object result;
        if (SqlCommandType.INSERT == this.command.getType()) {
            param = this.method.convertArgsToSqlCommandParam(args);
            result = this.rowCountResult(sqlSession.insert(this.command.getName(), param));
        }
        ...省.去.一.堆...
        else if (SqlCommandType.SELECT == this.command.getType()) {
            // 我们执行select会进入这里
            if (this.method.returnsVoid() && this.method.hasResultHandler()) {
                this.executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (this.method.returnsMany()) {
                result = this.executeForMany(sqlSession, args);
            } else if (this.method.returnsMap()) {
                result = this.executeForMap(sqlSession, args);
            } else if (this.method.returnsCursor()) {
                result = this.executeForCursor(sqlSession, args);
            } else {
                //这里就是处理参数的地方
                param = this.method.convertArgsToSqlCommandParam(args);
                // 然后进入这里，发现我们的方法，其实就是由sqlSession执行的
                result = sqlSession.selectOne(this.command.getName(), param);
            }
        } 
        ..省去一堆..

}
```

第三层：还是这个类中MapperMethod，进入这个方法`convertArgsToSqlCommandParam`

```java
public Object convertArgsToSqlCommandParam(Object[] args) {
    int paramCount = this.params.size();
    if (args != null && paramCount != 0) {
        if (!this.hasNamedParameters && paramCount == 1) {
            // 如果没有注解并且参数只有1个就不做处理
            return args[(Integer)this.params.keySet().iterator().next()];
        } else {
            // 多于1个或是有注解，就要开始封装map了！
            Map<String, Object> param = new MapperMethod.ParamMap();
            int i = 0;
			// 遍历 maps
            for(Iterator i$ = this.params.entrySet().iterator(); i$.hasNext(); ++i) {
                Entry<Integer, String> entry = (Entry)i$.next();
                param.put(entry.getValue(), args[(Integer)entry.getKey()]);
                String genericParamName = "param" + String.valueOf(i + 1);
                // 从param1开始计数作为key
                if (!param.containsKey(genericParamName)) {
                    param.put(genericParamName, args[(Integer)entry.getKey()]);
                }
            }

            return param;
        }
    } else {
        return null;
    }
}
```

`this.hasNamedParameters`这个值是一个boolean值，在构造时就会判断有没有加注解

### `#{}`与`${}`

Mybatis有两种取值的方法`#{}与${}`

- 都可以获取map中的值或pojo对象的值

区别`#{}`以预编译的形式，将参数设置到Sql语句中，类似于`PrepareStatement`，可以防止Sql注入

但是`${}`就是简单的将参数拼装在Sql中，会有安全问题

> 那还有`${}`干什么呢？

JDBC并不是所有操作都支持占位符，比如**分表操作、排序操作**，都只能使用`${}`

> `#{}`的进阶操作

在`#{}`还可以配置`javaType`、`jdbcType`、`mode`、`numericScale`、`resultType`、`typeHandler`、`jdbcTypeName`、`expression`

`javaType`用来标记当前字段的Java中的类型，一般无需使用

`mode`是存储过程中设置IN、OUT、INOUT

`jdbcType`可以设置对应数据库中的类型，当传入的值为NULL时，Mybatis会映射为OTHER，Mysql可以识别但是Oracle不可以，所以这里需要处理一下

```sql
#{email, jdbcType=OTHER}
```

### resultMap实现关联查询

比方说在`Employee`对象内，有一个成员为`Department`，这也是一个对象

显然自动映射是不可能帮我们实现这样的映射关系的，所以我们可以使用`resultMap`来实现

方式一：级联属性封装结果集

```xml
<resultMap>
    <id column="id" property="id"/>
    <result column="email" property="email"/>
    <result column="did" property="depart.id"/>
    <result column="dName" property="depart.name"/>
</resultMap>
```

方式二：使用`association`：可以实现一对一关联

- `property`：指定哪个属性为联合对象
- `javaType`：指定这个属性对象的类型（不可以省略）

```xml
<resultMap>
    <id column="id" property="id"/>
    <result column="email" property="email"/>
	<association property="dept" javaType="com.xx.xx.pojo.Department">
		<id column="did" property="id"/>
    	<result column="dName" property="departmentName"/>
    </association>
</resultMap>
```

方式三：使用`association`：进行**分步查询**，并在此之上**可以实现延迟加载**

> 延迟加载：我们一直用到了Employee，但不一定会用到Department，所以只有我们用到Department再去查Department，节省资源

```xml
<resultMap>
    <id column="id" property="id"/>
    <result column="email" property="email"/>
	<association 
                 property="dept"
                 select="com.xx.xx.dao.DepartmentMapper.getDepartmentById"
                 column="did"
                 >
        <!-- 基于分段查询可以实现延迟加载 -->
    </association>
</resultMap>
```

开启懒加载需要两步：

1. 在全局配置文件配置懒加载开启`lazyLoadingEnabled`为`true`
2. 在全局配置文件配置`aggressiveLazyLoading`为`false`

```xml
<settings>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

---

现在我们要去查一个部门下的所有员工，这属于一对多，就需要用到`collection`

- `property`：指定哪个属性为联合对象
- `ofType`：指定集合里面的对应元素的类型
- `select`：也可以实现延迟加载

```xml
<resultMap>
    <id column="did" property="id"/>
    <result column="dName" property="departmentName"/>
	<collection property="emps" ofType="com.xxx.xx.pojo.Employee">
        <id column="eid" property="id"/>
        <result column="name" property="name"/>
    </collection>
</resultMap>
```

## Mybatis缓存

Mybatis默认定义了**两级缓存**

- **一级缓存**：默认情况下只有一级缓存开启，是**SqlSession级别的缓存**，也称为**本地缓存**
- **二级缓存**：需要手动开启，是**namespace级别的缓存**，也叫**全局缓存**

为了提高扩展性，Mybatis也提供了缓存接口`Cache`，我们可以实现Cache来实现**自定义二级缓存**

> 缓存本质就是：一个`map`对象

### 一级缓存

> 一级缓存会存放：与数据库**同一次会话期间**查询到的数据

如果之后又用到了这些数据，会直接去缓存取值，但是一级缓存有四种情况，它会失效：

1. `sqlSession`不同：如果你查询同一个数据，但是**用了两个不同的`SqlSession`**，此时就会失效，因为一级缓存是`SqlSession`级别的
2. `sqlSession`相同，但是**查询条件不同**：（这是必然的）因为本地缓存没有
3. `sqlSession`相同，但是**两次查询期间，执行了增删操作！**：因为你的增删操作可能修改了我要查的值
4. `sqlSession`相同，但是**手动清除了一级缓存**：调用`session.clearCache()`，删除了一级缓存

### 二级缓存

基于namesapce级别的缓存，一个namespace对应一个二级缓存

> 二级缓存的工作机制：

1. 一个会话查询一个数据，查询完成后会将这个数据放在一级缓存中
2. 如果这个会话关闭，那么会将一级缓存的数据保存在二级缓存中
3. 不同的`namespace`指的就是不同的`Mapper`，比如说`EmployeeMapper`查询的数据会放在自己的Map中，`DepartmentMapper`查询的数据会放在它的Map中

> 二级缓存使用 

1. **开启二级缓存**（不同版本是否开启不一样，所以我们的全局配置最好显示配置一下）

```xml
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

2. **在mapper.xml中引入cache标签**

```xml
<cache eviction="" blocking="" readOnly="" flushInterval="" size="" type=" "></cache>
<!--
eviction：缓存的回收策略，有四种
		LRU（默认）、FIFO、SOFT（移除软引用对象）、WEAK（移除弱引用对象）
flushInterval：缓存刷新间隔，即缓存多长时间清空一次，默认为不清空，可以设置，单位为ms
readOnly：是否只读
	true：设置为只读，mybatis就会认为你不会进行修改，每次都会返回缓存中的值，
                  速度快，但是不安全
	false：设置为非只读，mybatis会觉得数据会被修改，会利用序列化与反序列化克隆一份新的数据给你
                  速度慢，但是安全，是默认的设置
size：缓存的大小
type：自定义缓存的全类名，实现cache接口，然后把实现类的全类名写在type内
-->
```

3. 由于cache默认设定为非只读，会用到序列化，所以**POJO需要实现序列化接口**

**这样就开启了二级缓存**

---

注意：

- 所有数据都会先放在一级缓存
- 如果SqlSession提交或者关闭之后，才会放入二级缓存

### 缓存有关的配置项

1、`cacheEnable=true`：全局配置文件中这一项配置的是二级缓存，对一级缓存没有影响

2、 每个select标签都有`useCache="true"`：这里也指的是二级缓存，设置为false，也不会影响一级缓存的使用

3、 每个增删改标签的`flushCache="true"`：增删改执行完成后就会清除缓存，会将一二级缓存全部清空

4、 `sqlSession.clearCache()`只是清除当前Session的一级缓存

5、全局配置文件中的`localCacheScope`：设置为statement可以禁用一级缓存

### 查询时缓存的顺序

> 当一个`SqlSession`去查询数据时，会**先查二级缓存**，然后**才去查一级缓存**，如果两级缓存都没有，才回去查数据库

![缓存工作原理](http://img.yesmylord.cn//image-20210913144004474.png)

