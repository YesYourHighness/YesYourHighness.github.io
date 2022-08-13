---
title: MySQL-2
date: 2019-08-31 11:22:07
tags: 
- MySQL
categories: 
- 后台
- MySQL
---

<center>
引言：

SQL的继续学习

DQL

四种约束
</center>

<!--more-->

----

# SQL

## DQL

### 基础查询

查询表中的记录
```sql
select 字段列表 
from 表名列表 
where 条件列表 
group by 分组字段 
having 分组之后的条件
order by 排序
limit 分页
```

1. 多字段查询
    ```sql
    SELECT age , `name` FROM stu;
    ```
    标准格式，还需要加上注释
    ```sql
    SELECT 
			age , -- 年龄
			`name`  -- 姓名
    FROM 
			stu;  -- 学生表
    ```
2. 去除重复
    ```sql
    SELECT DISTINCT sex FROM stu;
    # 加一个DISTINCT关键字即可
    # 没能去除可能是因为多了一个空格或者回车
    SELECT DISTINCT sex , 'name' FROM stu;
    # 多表查询时只有被查询项都相同才会去重
    ```

3. 计算列
    ```sql
    SELECT sex,id + age FROM stu;
    # 如果有NULL数据就要使用IFNULL来吧NULL变为0
    ```
4. 起别名

    ```sql
    SELECT sex,id + age AS 总和 FROM stu;
    # 别名可以起名为中文，但是不推荐
    # AS 可以不写， 用空格来替代
    ```
### 条件查询
SQL的运算符：
`>` `<` `<=` `>=` `= ` `<>或!=`(不等于)

`BETWEEN ... AND`

`IN` 

`LIKE`模糊查询

`IS NULL`

`and 或 &&` `or 或 ||` `not 或 |`

语法
```
WHERE 条件
```
示例
```sql
SELECT * FROM stu WHERE age >50; # 大于
SELECT * FROM stu WHERE age <> 99; # 不等于
SELECT * FROM stu WHERE age != 99; # 不等于
SELECT * FROM stu WHERE age BETWEEN 10 AND 50; # 区间

SELECT * FROM stu WHERE age = 18 OR age = 22 OR age = 20; # 或者
SELECT * FROM stu WHERE age IN (18,22,20); # 简便的或者

SELECT * FROM stu WHERE age  =  NULL; # NULL是不可以这样判断的
SELECT * FROM stu WHERE age IS NULL; # 正确方法
SELECT * FROM stu WHERE age IS NOT NULL; # 正确方法
```

### 模糊查询
`LIKE`关键字

占位符
```sql
%  多个任意字符
_  单个任意字符
```
示例
```sql
# 查询姓马的人
SELECT * FROM stu WHERE `name` LIKE '马%';
# 查询第二个字是化的人
SELECT * FROM stu WHERE `name` LIKE '马%';
# 查询名字是三个的人
SELECT * FROM stu WHERE `name` LIKE '___';
# 查询含有马字的人
SELECT * FROM stu WHERE `name` LIKE '%马%';
```

### 排序查询


语法
```sql
order by 子句
order by 排序字段1 排序方式1，排序字段2 排序方式二
第二排序条件只有当第一排序条件一致时才会发挥作用
```
默认为从小到大排序

排序方式共两种
```sql
ASC //升序
DESC //降序
```
示例
```sql
SELECT * FROM `stu` ORDER BY age ASC,qq asc
```

### 聚合函数
将一列数据作为一个整体，进行纵向的计算
1. count 计算个数
    - **一般选择非空的列，主键**
    - **COUNT(*)也可以，但不推荐**
2. max 计算最大值
3. min 计算最小值
4. sum 计算和
5. avg 计算平均值

注意：**排除NULL值**


示例
```sql
SELECT COUNT(NAME) FROM stu;
SELECT SUM(qq) FROM stu;

SELECT COUNT(IFNULL(qq,0)) FROM stu;#如果qq是null，那就换成0
```

### 分组查询

语法：
```
group by 分组字段
```
注意：
1. 分组之后查询的字段**必须是该分组字段或者聚合函数**
   
    ```sql
    SELECT sex,SUM(qq) FROM `stu` GROUP BY sex
    
    SELECT sex,MIN(qq) 
    FROM `stu` 
    WHERE qq > 5000 
    GROUP BY sex
    # 可以在分组之前加条件
    
    SELECT sex,MIN(qq) 
    FROM `stu` 
    WHERE qq > 5000 
    GROUP BY sex 
    HAVING sex = '男';
    # 分组之后还可以继续添加条件
    ```
2. `HAVING`和`WHERE`语句的区别
    - `WHERE`作用在分组前，`WHERE`后不可以跟聚合函数
    - `HAVING`作用在分组之后,`HAVING`可以使用聚合函数


### 分页查询

语法：
```
limit 开始的索引,每页查询的条数;
```
分页操作在Oracle,MySQL内是不一样的

示例代码
```sql
SELECT * FROM stu LIMIT 0,2; -- 第一页
SELECT * FROM stu LIMIT 1,2 ;-- 第二页

# 公式：开始的索引 = （当前页码 -1）* 每页的条数
```

## 约束

对表中的数据进行限定，保证数据的正确性，有效性和完整性

分类：
```
1. 主键约束 primary key
2. 非空约束 not null
3. 唯一约束 unique
4. 外键约束 foreign key
```

### 非空约束

值不能为空

`not null`
语法：

- 创建表时添加约束
```sql
CREATE TABLE student(
	id INT,
	NAME VARCHAR(20) NOT NULL
)
# 此后添加数据NAME则为必填项
```
- 普通添加约束
```sql
ALTER TABLE student MODIFY NAME VARCHAR(20) NOT NULL;
# 覆盖掉原有
```

- 删除非空约束
```sql
ALTER TABLE student MODIFY NAME VARCHAR(20);
# 覆盖掉原有的not null
```

### 唯一约束
值不能重复

`unique`语法

- 创建表时添加约束
```sql
CREATE TABLE stu(
	id INT,
	number VARCHAR(20) UNIQUE; # 添加唯一约束
)
# 此后number是不可以重复的值
# 唯一约束的限定值可以有多个null
```
- 普通添加约束
```sql
ALTER TABLE stu MODIFY number VARCHAR(20) UNIQUE;
```

- 删除约束
```sql
ALTER TABLE stu DROP INDEX number;
```

### 主键约束
注意：
1. **非空且唯一**
2. 一张表只能有一个字段为主键
3. 主键就是表中记录的唯一标识

语法：
- 创建表时添加约束
```sql
CREATE TABLE student (
	id INT PRIMARY KEY,
	NAME VARCHAR(20)
)
```
- 普通添加约束
```sql
ALTER TABLE stu MODIFY id INT PRIMARY KEY;
```

- 删除主键
```sql
ALTER TABLE stu DROP PRIMARY KEY
```
- 主键约束的自动增长

如果某一列是数值类型的，使用关键字`auto_increment`可以完成值的自动增长

```sql
CREATE TABLE x (
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20)
)
```
- 删除自动增长
```sql
ALTER TABLE stu MODIFY id INT;
```
- 添加自动增长
```sql
ALTER TABLE stu MODIFY id INT AUTO_INCREMENT;
```

### 外键约束

保证表中数据的一些逻辑完整性，让表与表产生一些关系

**外键值可以为null，但是不可以为不存在的值**

1. 创建表时添加外键
```sql
CREATE TABLE 表名(
    ...
    外键列
    CONSTRAINT 外键名称 FOREIGN KEY 外键列名称 REFERENCES 主表名称(主列表名称)
);
```

2. 普通添加外键
```sql
ALTER TABLE 表名 ADD CONSTRAINT 外键名 FOREIGIN KEY (外键字段) REFERENCES 主表名称(主表列名称)
```
3. 删除外键
```sql
ALTER TABLE 表名 DROP FOREIGIN KEY 外键名
```
代码示例
```sql
# 创建部门表
CREATE TABLE department (
	id INT PRIMARY KEY AUTO_INCREMENT,
	dep_name VARCHAR(20),
	dep_locationion VARCHAR(20)
);
# 导入数据
INSERT INTO department(id,dep_name,dep_locationion) VALUES(NULL,'销售部','深圳');
INSERT INTO department(id,dep_name,dep_locationion) VALUES(NULL,'研发部','广州');
# 创建员工表并创建外键
CREATE TABLE emp (
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(30),
	age INT,
	dep_id INT,
	CONSTRAINT emp_dept_fk FOREIGN KEY (dep_id) REFERENCES department(id)
);
# 导入数据
INSERT INTO emp(NAME,age,dep_id) VALUES('张三',20,1);
INSERT INTO emp(NAME,age,dep_id) VALUES('李四',20,2);
INSERT INTO emp(NAME,age,dep_id) VALUES('王五',20,3);
```

4. 级联操作
- 添加级联
```sql
# 创建表之后
ALTER TABLE 表名 ADD CONSTRAINT 外键名称
FOREIGIN KEY (外键字段名) REFERENCES 主表名称 ON UPDATE CASCADE ON DELETE CASCADE
# 最后三个单词根据操作来更改
```
- 分类
    - 级联更新 : `ON UPDATE CASCADE`
    - 级联删除 : `ON DELETE CASCADE`

- 好处：方便
- 缺点：会把有联系的数据全部删除，十分危险