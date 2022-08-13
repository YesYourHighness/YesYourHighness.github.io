---
title: Mysql实战
date: 2021-09-05 19:39:53
tags: 
- MySQL
categories: 
- 后台
- MySQL
---

<center>
引言：Mysql实战；实操环节~
</center>
<!--more-->

# Mysql实战

## Mysql中数据类型括号中数字表示的含义

### 字节与字符

字节与字符是什么就不说了，这里主要说一下Mysql中的字符与字节

1. **UTF-8编码**：一个英文字符等于**一个字节**，一个中文（含繁体）字符等于**三个字节**

2. **Unicode编码**：一个英文等于**两个字节**，一个中文（含繁体）字符等于**两个字节**

   符号：**英文标点占一个字节，中文标点占两个字节**

   举例：英文句号`.`占1个字节的大小，中文句号`。`占2个字节的大小。

3. **UTF-16编码**：一个英文字母字符或一个汉字字符存储都需要2个字节（Unicode扩展区的一些汉字存储需要4个字节）

4. **UTF-32编码**：世界上任何字符的存储都需要4个字节。

> utf-8与utf-8mb4的区别

如果要存互联网emoji表情，例如昵称，聊天，**就需要utf8mb4，而不是utf-8**

**MySQL数据库的 “utf8”并不是真正概念里的 UTF-8。**

MySQL中的“utf8”编码只支持最大3字节每字符。

真正的大家正在使用的UTF-8编码是应该能支持4字节每个字符。

MySQL中的 “utf8mb4” 才是 真正意义上的“UTF-8”。

### char与varchar

> `char(8)`与`varchar(8)`表示什么意思？

- `char`的长度可选范围在0-255**字符**之间（注意单位是字符）

- `varchar`的长度是可变的，mysql5.0.3之后varchar的长度范围为0-65535个字节（注意这里是字节）

所以：

1. `char(8)`就表示char的长度为8，可存储8个**字符**，如果你存储的少于8个字符，它会自动在右方填充空格补齐
2. `varchar(8)`可变大小，最大为8；它的长度为其实际长度+1，多出来的1是因为varchar需要存储长度这个信息

varchar(m)里面表示的是长度，例如：varchar（5）表示最大可存储5个中文或5个英文字母。 

### int系列

| 类型      | 大小 | 范围（有符号）                                          | 范围（无符号）                  | 用途       |
| --------- | ---- | ------------------------------------------------------- | ------------------------------- | ---------- |
| TINYINT   | 1B   | （-128，127）                                           | （0，255）                      | 小整数值   |
| SMALLINT  | 2B   | （-32 768,32 767）                                      | （0，65535）                    | 大整数值   |
| MEDIUMINT | 3B   | （-8 388 608，8 388 607）                               | (0，16 777 215)                 | 大整数值   |
| INT       | 4B   | (-2 147 483 648，2 147 483 647) (0，4 294 967 295)      | (0，4 294 967 295)              | 大整数值   |
| BIGINT    | 8B   | (-9 233 372 036 854 775 808，9 223 372 036 854 775 807) | (0，18 446 744 073 709 551 615) | 极大整数值 |

int系列的大小，是固定的，那么`int(8)`代表什么呢？

8代表显示的长度，如果你开启了**零填充**，存储了一个1，那么此时你查到的数据为`0000 0001`

## 默认创建的数据库

（这是使用Navicat生成的数据库，转储后的sql内容）

```sql
DROP TABLE IF EXISTS `player`;
CREATE TABLE `player`  (
  `player_id` int(11) NOT NULL AUTO_INCREMENT, # AUTO_INCREMENT自增
  `team_id` int(11) NOT NULL,
  `player_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `height` float(3, 2) NULL DEFAULT NULL,
  PRIMARY KEY (`player_id`) USING BTREE,
  UNIQUE INDEX `player_name`(`player_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
# 默认使用Innodb存储引擎
# 默认编码为utf8
# 默认排序方式为utf8_general_ci，这是一个大小写不敏感的排序方式
# utf8_bin这个排序方式对大小写敏感
# 默认行格式为Dynamic
```

## ID字段【待补充】









