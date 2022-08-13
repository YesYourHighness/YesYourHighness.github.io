---
title: MySQL的特殊查询
date: 2021-09-02 21:46:53
tags: 
- MySQL
categories: 
- 后台
- MySQL

---

<center>
引言：Mysql的特殊查询
</center>

<!--more-->

# Mysql独有的limit

```sql
select * from tableName limit i,n
# tableName：表名
# i：为查询结果的索引值(默认从0开始)，当i=0时可省略i
# n：为查询结果返回的数量
# i与n之间使用英文逗号","隔开

limit n 等同于 limit 0,n
```

使用`limit`去实现TOP k问题

```sql
SELECT * FROM 表 order by 字段 desc limit k;
# 可以查询前k个记录
SELECT * FROM ql_article ORDER BY hits DESC LIMIT 1,3;
# 查询第二页（注意是从0开始的），前三条记录
```

举个例子：

![查询图](http://img.yesmylord.cn//image-20210902143903422.png)

```sql
# 1、查询蓝框内容，就是前三条
SELECT * FROM ql_article LIMIT 3;
# 相当于这么写
SELECT * FROM ql_article LIMIT 0, 3;
# 2、查询红框内容
SELECT * FROM ql_article LIMIT 3, 3;
# 跳过前三条记录，取三条数据
# 3、查询黄框内容
SELECT * FROM ql_article LIMIT 8, 4;
```

# 连接查询

有四种连接

- 左外连接`left join`
- 右外连接`right join`
- 内连接`inner join`
- 全连接`UNION`（注意，和Oracle不同的！）

区别要记住：

1. **左外连接中优先匹配左表，右表没有的显示为null**
2. 右外连接是，右表全部有，左表可能为空；
3. 内连接中均不能为空；
4. 全连接都会匹配；

基本语句为`select xxxx from x1 left join x2 on x1.xx = x2.xx;`（即记住`表1 left join 表2 on 条件`）

优点枯燥，这里用栗子解释一下：

```sql
select ql_article.uid, ql_user.username
from ql_article 
LEFT JOIN ql_user 
ON ql_article.uid = ql_user.uid;
# 将写了文章的用户，查出他们的名字
```

结果：

![左连接右表可能为null](http://img.yesmylord.cn//image-20210902145953507.png)

全连接的语句有点特殊：

```sql
select uid, null
from ql_article 
UNION select uid,ql_user.username
from ql_user;
# 它会自动将你相同的键合在一起
```

注意：

- **UNION左右两个查询的列个数必须一样**

- 如果要排序请写在外面：

  ```sql
  (select id,name from A order by id) union all (select id,name from B order by id);
  #没有排序效果
  
  (select id,name from A ) union all (select id,name from B ) order by id;
  #有排序效果
  ```

  