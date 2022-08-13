---
title: Python序列
date: 2021-05-01 22:47:21
tags:
- Python
categories: 
- Python
---
<center>
引言：

Python中的序列：列表、元组、字符串
</center>

<!--more-->

# Python序列

## python序列

序列：分为两种
- 有序序列：列表、元组、字符串、（range、zip、map、enumerate等）
- 无序序列：集合、字典

也可以这么分：
- 可变序列：列表、集合、字典
- 不可变序列：元组、字符串、（range、zip、map、enumerate等）

通常称序列就是有序序列，这里我们只讨论有序序列

> 序列：Python中最基本的数据结构
>
> 序列中的每个元素都分配一个数字 ——它的位置，或索引，第一个索引是0，第二个索引是1，依此类推

### 六个内置序列

python有六个内置序列

1. 列表
2. 元组
3. 字符串
4. Unicode字符串（简单归在字符串中讨论）
5. buffer对象（不讨论）
6. xrange对象（不讨论）



## 列表

列表是最常用的Python数据类型，它可以作为一个方括号内的逗号分隔值出现。

要点：

1. 列表存储的内容不需要具有相同的数据类型
2. 列表支持切片操作（切片有基本的操作和较高阶的操作）
3. 区别列表的添加方法`append`、`extend`、`insert`
4. 列表的删除，注意区别`remove`和`pop`的区别
5. 列表的脚本操作

### 列表存储的内容不需要具有相同的数据类型

```python
# 1. 列表的存储的数据不需要相同的数据类型
arr = [1, 2, 3, 4, "字符串", ["另一个列表"]]
```

### 列表支持切片操作

序列此处的切片方法类似，其他的序列将不再详细叙述切片操作

```python
# 2. 通过切片方式访问列表
''' 切片操作
list[a:b:c]可以传入一个、两个、或三个参数
一个参数：直接显示对应index的元素。
        注意：正数从0开始，从左向右；负数从-1开始，从右向左
两个参数：按两个参数对应的index截取出一个list。
        注意：切片为左闭右开区间，即[a,b)，index为b的元素是不在其中的
三个参数：第三个参数代表步进，默认为1，如果为负数，代表从右向左
'''
arr = [1, 2, 3, 4, "字符串", ["另一个列表"]]
# 2.1 切片基本操作
print(arr[:])  # 都不传表示整个列表
print(arr[::-1])  # 逆向打印整个列表
print(arr[3])  # 显示第四个元素，即4
print(arr[-3])  # 显示倒数第三个元素，即4
print(arr[1:4])  # 打印出[1,4)的内容，即[2, 3, 4]
print(arr[1:4:2])  # 隔一个打印一个，打印出[2, 4]

# 2.2 高阶操作
print(arr[:4])  # 第一个参数省略，默认为0
print(arr[1:])  # 第二个参数省略，默认为列表的长度
print(arr[::2])  # 获取下标1,3,5...
print(arr[1::2])  # 获取下标2,4,6...

print(arr[1:3:-2])  # 打印出[]
print(arr[3:1:-2])  # 打印出[4]
'''
以上两个为什么是这样呢？
我们可以这样理解：
1. arr[1:3:-2] 
    其中1,3代表获得[1,3)的数据，是从左向右的，但是-2又是从右向左，方向不一致，因此打印的结果为空列表
2. arr[3:1:-2]
    其中3:1 从右向左获得[3,1)的数据，-2也是从右向左，方向一致，所以不为空
'''
print(arr[3:1])  # 打印出[]，因为第三个参数默认为1
print(arr[1:100])  # 第二个参数可以超过列表的长度
print(arr[100:4:-1])  # 打印出[['另一个列表']]
```

### 区别列表的添加方法`append`、`extend`、`insert`

```python
# 3. 列表的添加，区别 append、extend、insert
# 3.1 append函数
arr.append("添加")  # append添加一个元素
print(arr)  # 打印出[1, 2, 3, 4, '字符串', ['另一个列表'], '添加']

res = arr.append("会有返回值吗？")
print(res)  # 打印出None，append方法没有返回值，只会修改原有的列表

newArr = [985, 211]
arr.append(newArr)  # 可以添加新的列表，注意：是把这个列表当做一个元素，添加到原有的列表中
print(arr)
# 打印如下：[1, 2, 3, 4, '字符串', ['另一个列表'], '添加', '会有返回值吗？', [985, 211]]

# 3.2 extend
newArr = [985, 211]
print(len(arr))  # 9
res = arr.extend(newArr)
print(len(arr))  # 11，注意extend是将两个列表合并
print(res)  # 也是None 没有返回值
# 也可以用+来替代extend
arr = arr + newArr
# 注意：其实，+是返回的一个新的列表，不是原地操作。+=才与extend相同，均为原地操作，高效
print(len(arr))  # 13

# 3.3 insert
newArr = [985, 211]
res = newArr.insert(0, "双一流")
print(newArr)  # 插入到第0个位置之前 ['双一流', 985, 211]
# 注意我这里的说法，插入到第0个元素之前，之前！！
print(res)  # 也没有返回值

newArr.insert(-1, "C9")
print(newArr)  # ['双一流', 985, 'C9', 211]
# 如果你记住了我说的插入到第x个元素之前，这里就好理解了
```

总结一下三种添加操作的区别：

| 函数         | append             | extend                                             | insert                        |
| ------------ | ------------------ | -------------------------------------------------- | ----------------------------- |
| 参数         | 一个：要加入的元素 | 一个：参数必须**可迭代**（例如列表、元组、字符串） | 两个：一个index，一个元素的值 |
| 添加数组     | 作为一个元素添加   | 合并两个数组                                       | 不能                          |
| 是否有返回值 | 无                 | 无                                                 | 无                            |



### 列表的删除，注意区别`remove`和`pop`的区别

```python
# 4. 列表的删除，注意区别remove和pop的区别
newArr = [985, 211, "C9", "Top2", "双非"]
# 4.1 pop函数
res = newArr.pop()  # 默认删除最后一个元素
print(newArr)  # 打印出[985, 211, 'C9', 'Top2']
print(res)  # 有返回值，返回被删除的元素 双非

newArr.pop(1)  # 有一个参数，这个参数是index
print(newArr)  # [985, 'C9', 'Top2']

# newArr.pop(100) 报错，参数index不能超过列表范围

# 4.2 remove函数
res = newArr.remove("C9")
print(res)  # None remove函数没有返回值

# newArr.remove("不存在的元素") 不存在这个元素，会报错
# 结合 in 使用：
# if x in newArr:
#     newArr.remove(x)
```

区别一下两个删除操作：

| 函数   | pop                             | remove                                     |
| ------ | ------------------------------- | ------------------------------------------ |
| 参数   | 一个：index，默认为最后一个下标 | 一个：要删除的值，如果列表没有这个值，报错 |
| 返回值 | 有，被删除的元素                | 无                                         |



### 列表的脚本操作

```python
# 5 列表的脚本操作符
# 5.1 求长度
len(arr)
# 5.2 组合：以下这两种方法一样的功能
arr = arr + newArr
arr.extend(newArr)
# 5.3 重复
newArr = [1]
newArr *= 5  # 重复五次
print(newArr)  # [1, 1, 1, 1, 1]
# 5.4 in判断是否存在
res = 1 in newArr
print(res)  # True
# 5.5 迭代操作，列表可以作为迭代的范围
newArr = [1, 2, 3, 4, 5]
for i in newArr:
    print(i)
# 注意不能这样使用
# for i in newArr:
#     print(newArr[i]) 这里的i是列表内的每一个元素
# 也可以这么用
for i in range(len(newArr)):
    print(newArr[i])
```



## 元组

> Python的元组与列表类似，不同之处在于元组的元素不能修改。
>
> 元组使用小括号，列表使用方括号。
>
> 元组创建很简单，只需要在括号中添加元素，并使用逗号隔开即可。

要点：

1. 元组的创建，尤其注意**只有一个元素的情况**，末尾要加逗号
2. 元组的访问
3. 元组**不允许修改值**，但是元组允许拼接
4. 元组**不允许删除元素**，只允许删除整个元组
5. 元组的脚本运算
6. 任意无符号的对象，以逗号隔开，默认为元组

### 元组创建

```python
# 1. 元组的声明
tup = ()  # 空元组
tup = (1)  # 这样是 tup 是1，不是元组，下面才对
tup = (1,)  # 元组中只包含一个元素时，需要在元素后面添加逗号
tup = (1, 2, 3, "你好", [1, 2])
```

### 元组访问

```python
# 2. 元组的访问，同列表，这里不再多次叙述
tup[0]
tup[::-1]  # 逆序
```

### 元组拼接

```python
# 3. 元组的拼接
tup1 = (1, 3)
tup2 = (2, 4)
print(tup2 + tup1)  # 打印出：(2, 4, 1, 3)
```

### 元组删除

```python
# 4. 元组的删除
# 注意：元组中的元素值是不允许删除的，但我们可以使用del语句来删除整个元组
del tup
```

### 元组脚本操作

```python
# 5. 元组的脚本运算
tup = (1, 2, 3, "你好", [1, 2])
# 5.1 长度
len(tup)
# 5.2 合并操作
tup = tup + tup1
# 5.3 判断元素是否存在
res = 3 in tup
print(res)  # True
# 5.4 重复
(1,)*5 # 重复五次
# 5.5 迭代操作
for i in tup:
    print(i)
```

### 无关闭分隔符

```python
# 6. 无关闭分隔符
# 任意无符号的对象，以逗号隔开，默认为元组
# print 'abc', -4.24e93, 18+6.6j, 'xyz' Python2中print支持不带括号，Python3必须带括号
x, y = 1, 2  # 经常使用的交换元素的值默认就是元组的操作
# 通过元组按照index进行赋值的方式进行重新赋值给两个变量
print(x, y, sep=" ")
```



## 字符串

字符串是 Python 中最常用的数据类型。我们可以使用引号(`'`或`"`)来创建字符串。

本节过于基础，不再进行多余叙述

### 基础内容

```python
# 1. 字符串的创建
str = 'a'  # python中不存在单字符类型，这也是一个字符串
str = "你好"

# 2. 字符串访问，支持切片，不再叙述
print(str[0])
print(str[::-1])  # 逆序

# 3. 字符串连接
str1 = "你"
str2 = "好"
print(str1 + str2)

# 4. 字符串in判断
str = "abcDefg"
print("b" in str)  # True
print("d" in str)  # False

# 5. 字符串三引号
# Python 中三引号可以将复杂的字符串进行赋值。
errHTML = '''
<HTML><HEAD><TITLE>
Friends CGI Demo</TITLE></HEAD>
<BODY><H3>ERROR</H3>
<B>%s</B><P>
<FORM><INPUT TYPE=button VALUE=Back
ONCLICK="window.history.back()"></FORM>
</BODY></HTML>
'''
# 6. Unicode 字符串
uStr = u"Unicode String"
print(uStr)
uStr = u'Hello\u0020World !'  # \u0020 是空格的Unicode编码，十六进制
print(uStr)  # 打印出Hello World !

print(ord(" "))  # ord函数返回对应的 ASCII 数值，这里返回空格的值为32
print(chr(97))  # chr函数是ord函数的配对函数，输入ASCII数值，返回字符，这里返回a
```



### 内置函数

```python
# 7. 字符串内置函数
# 字符串最重要的内容，就是掌握使用字符串函数
# 以下只展示比较重要的内置函数
str = "abcDa Bye"
print(str.capitalize())  # Abcda bye 如果第一个字符是小写字母，将它变为大写，其他字符变为小写
print(str.count("b"))  # 查询出现的次数，这里返回1
print("第一个数{}第二个数{}".format(3, 4))  # 第一个数3第二个数4
print("第二个数{1}第一个数{0}".format(1, 2))  # 第二个数2第一个数1，format函数可以设置位置
print(str.index("a"))  # 返回该字符第一次出现时的下标
print(",".join(str))  # a,b,c,D,a, ,B,y,e 按前面的样子分隔后面的字符串
print(str.lower())  # abcda bye
print(str.upper())  # ABCDA BYE
print(str.swapcase())  # 翻转大小写 ABCdA bYE
# max、min加字符串，返回对应ascii码大的或小的元素
print(max(str))  # 返回 y
print(min(str))  # 返回一个空格
# 替换
print(str.replace("a", "A"))  # AbcDA Bye 替换，第一个参数为老字符串，第二个参数为新字符串
# 删除空格
print("  str  ".strip())  # 删除字符串两边的空格
```





