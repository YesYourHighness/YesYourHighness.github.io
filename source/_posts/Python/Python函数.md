---
title: Python函数
date: 2021-05-02 21:23:33
tags: 
- Python
categories: 
- Python
---
<center>
引言：

Python函数
</center>

<!--more-->

# Python函数

## Python函数



### 函数基础

```python
# 1. 函数的定义
'''
语法：
def name(args):
    body
    return ret
'''

def sayHi():
    print('hello!')

# 2. 函数调用：使用函数名加括号即可
sayHi()
print(sayHi) # <function sayHi at 0x00000277B1E961F0>
```

### 参数传递后是否可以被可变

在 python 中，`strings`,`tuples`, 和 `numbers`是不可更改的对象，而 `list`,`dict` 等则是可以修改的对象。

- **不可变类型：**变量赋值 **a=5** 后再赋值 **a=10**，这里实际是**新生成**一个 `int` 值对象 10，再让 a 指向它，而 5 被丢弃，不是改变a的值，相当于新生成了a。
- **可变类型：**变量赋值 **la=[1,2,3,4]** 后再赋值 **la[2]=5** 则是将 列表`la` 的第三个元素值更改，本身`la`没有动，只是其内部的一部分值被修改了

由此，python的参数传递：

对于不可变类型，传递值（类似值传递）：改变值不会影响原有的参数

对于可变类型，传递引用（类似引用传递）：改变值，原有的值也会改变

```python
# 3. 参数传递
# 3.1 不可变类型
def changeType1(a):
    a = '你好'
    print(a)  # 打印 你好

x = 1
changeType1(x)
print(x)  # 打印 1，可见x的值没有改变


# 3.2 可变类型
def changeType2(a):
    for i in range(len(a)):
        a[i] = 1

arr = [1, 2, 3, 4]
changeType2(arr)
print(arr)  # 打印出：[1, 1, 1, 1]

# 注意！！这种迭代将不会改变原有的值
def changeType3(a):
    for i in a:
        i = 1

arr = [1, 2, 3, 4]
changeType3(arr)
print(arr)  # 依然是：[1, 2, 3, 4]
```

### 参数类型

分为四类：

1. 必备参数：须以正确的顺序传入函数。调用时的数量必须和声明时的一样。
2. 关键字参数：函数调用可以**使用关键字参数**来确定传入的参数值
3. 默认参数：默认参数的值如果没有传入，则被认为是默认值
4. 不定长参数：

下面我们依次给出例子：

```python
# 4. 参数类型
# 4.1 必备参数
def sum(a, b):
    return a + b


# print(sum()) 报错
# print(sum(1)) 报错
print(sum(1, 3))  # 成功 返回4


# 4.2 关键字参数
def devide(a, b):
    return a / b


print(devide(1, 2))  # 0.5 默认按顺序
print(devide(b=1, a=2))  # 2.0 可以使用关键字指定参数输入


# 4.3 默认参数
def multiply(a=10, b=5):
    return a * b

print(multiply())  # 50 没有输入参数，全部默认
print(multiply(1))  # 5 输入一个参数，另一个默认
print(multiply(1, 9))  # 9 输入参数

# 4.4 不定长参数
def sumplus(*a): # 可变参数的标志，加一个*符号
    res = 0
    # 把可变参数当做列表使用即可
    for i in a:
        res += i
    return res
print(sumplus()) # 0
print(sumplus(1,2,3,4,5,6,7,8,9)) # 45
```

### 匿名函数

python中的匿名函数要点：

1. 只能包含一个表达式
2. 内部不能访问参数列表外的或是全局变量

python中的lambda表达式更多用在很多方法里面，例如`sort`方法：

```python
# 5. 匿名函数（lambda函数）
'''
语法：
lambda [arg1 [,arg2,.....argn]]:expression
'''
sumLambda = lambda a=1, b=2: a + b
print(sumLambda(1, 2))  # 3
print(sumLambda())  # 3
print(sumLambda) # <function <lambda> at 0x000001D8D3630EE0>
```

### 全局变量与局部变量

```python
# 6. 全局变量和局部变量
# 6.1 基本使用
num = 10 # 这就是一个全局变量
def var1():
    print(num)
var1() # 打印出 10

def var2():
    num = 1
    print(num)# 因为局部变量存在，所以自动屏蔽全局变量
var2() # 打印出 1

# 6.2 global变量的使用
# 函数内部定义的变量只在函数体内有效
def var3():
    cat = "小黄猫"
#print(cat)  报错：NameError: name 'cat' is not defined
# 但是可以使用关键字global来声明变量
def var4():
    # global cat="小黄猫" 注意声明和赋值必须分开，这样使用不正确
    global cat
    cat = "小黄猫"
var4() # 注意要先调用这个函数，才会产生那个全局变量
print(cat) # 打印出 小黄猫

# 局部变量和全局变量混合使用
dog = "史努比"
def var5():
    # print(dog) SyntaxError: name 'dog' is used prior to global declaration
    # 在全局变量声明前，不能使用这个变量
    global dog
    dog = "史迪奇"
    print(dog) # 史迪奇
var5()
print(dog) # 史迪奇 ，可以看到 dog全局变量已经被改变

# del dog 可以删除全局变量
```

## 要点总结

python函数的使用比较简单，但是要注意几个地方：

1. 函数名+括号()，代表调用这个函数，只使用函数名仅仅只代表一个地址
2. python参数不需要指定类型
3. `return`设置返回值，不写默认返回`None`
4. 传参时按默认顺序，可以使用关键字改变参数顺序
5. 可变参数使用符号`*`，使用时使用`for i in var`使用即可
6. 全局变量使用`global`声明、`del`删除