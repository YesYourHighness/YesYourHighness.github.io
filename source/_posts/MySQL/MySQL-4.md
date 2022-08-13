---
title: MySQL-4
date: 2019-08-31 21:46:50
tags: 
- MySQL
categories: 
- 后台
- MySQL
---

<center>
引言: 多表查询
</center>

<!--more-->




# 多表查询

查询语法
```sql
SELECT
    列表名称    
FROM
    表单名称
WHERE
    条件
```

示例：
```sql
SELECT * FROM emp,department;
# 会从两个表中组合所有情况显示为一张表（笛卡尔积）
# 例如A表有两种情况，B表有三种情况，结果就有2*3六种情况
```
这并不是我们想要的情况，我们需要淘汰那些有毛病的数据

## 多表查询的分类
### 内连接查询
```sql
- 隐式内连接：使用`WHERE`来消除无用的数据
- 显式内连接：`SELECT 字段列表 FROM 表名1 INNER JOIN 表名2 ON 条件`
```

隐式
```sql
SELECT
	t1.NAME, # 员工姓名
	t1.age, # 员工年龄 
	t2.dep_name # 部门名称
FROM
	emp t1,
	department t2
WHERE
	t1.dep_id = t2.id
```
显示
```sql
SELECT * 
FROM emp JOIN department ON emp.dep_id = department.id
# INNER 可以省略
```

### 外连接查询
- 左外连接：左表所有数据及其交集
- 右外连接：右表所有数据及其连接

语法
```sql
SELECT 字段列表 
FROM 表1 
LEFT OUTER JOIN 表2 ON 条件
# OUTER可以省略
```
示例代码
```sql
SELECT t1.*,t2.dep_name FROM emp t1 LEFT JOIN department t2 ON t1.dep_id=t2.id
# LEFT和RIGHT可以选择
# 可以主要显示一个表，副显示另一个表，不管另一个表的对应值是否存在
```


### 子查询

查询中嵌套查询
```sql
SELECT MAX(salary) FROM emp;
# 从员工表中查询最高工资,假设为9000
SELECT * FROM emp WHERE emp.salary = 9000;
# 查询工资为9000的人
```
一条查询即可完成
```sql
SELECT * 
FROM emp 
WHERE emp.salary = (SELECT MAX(salary) FROM emp);
```

#### 子查询的不同情况

1. 子查询结果是**单行单列**的
    - 子查询可以作为条件，可以使用运算符计算`> < <= >= =`
2. 子查询的结果是**多行单列**的
    - 子查询可以作为条件，可以使用运算符计算`IN`
3. 子查询的结果是**多行多列**的
    - 放入`FROM`中

