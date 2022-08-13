---
title: StringJoiner
date: 2020-02-24 21:52:59
tags:
- Java核心类
- StringJoiner
categories:
- Java核心类
- StringJoiner
---

<center>
    引言：StringJoiner
</center>
<!--more-->

# StringJoiner

为了更更更进一步高效的拼接字符串，Java1.8出现了
`StringJoiner`类

---

现在有一个字符串数组，里面有几个人名，我们现在要实现`你好：小明，小红，小王！`这样的效果
```java
String[] str = {"小明","小红","小王"};
StringBuilder sb = new StringBuilder("你好：");
//逐个加上符号
for (String s : str) {
    sb.append(s+",");
}
//删除最后一个多余的，
sb.delete(sb.length()-1,sb.length());
sb.append("!");
System.out.println(sb);//你好：小明,小红,小王!
```
感觉很繁琐，很琐碎，但是使用`StringJoiner`不一样了

```java
StringJoiner sj = new StringJoiner(",","你好：","!");
String[] str = {"小明","小红","小王"};
for (String s:str) {
    sj.add(s);
}
System.out.println(sj);//你好：小明,小红,小王!
```
简单方便，已经处理好了，不需要处理细节

## 构造方法
一共有
```java
StringJoiner(CharSequence delimiter)
//以分隔符构造一个实例
StringJoiner(CharSequence delimiter, CharSequence prefix, CharSequence suffix)
/*三个参数分别是：
1. 分隔符
2. 字首
3. 字尾*/
```
## 常用方法
`StringJoiner`底层其实也是用`StringBuilder`类实现的

```java
public StringJoiner setEmptyValue(CharSequence emptyValue)
/*设置确定此StringJoiner的字符串表示形式并且尚未添加任何元素（即，当它为空时）时要使用的字符序列。
为此，复制了emptyValue参数。
请注意，一旦调用了add方法，即使添加的元素对应于空字符串，StringJoiner也不再被认为是空的。
*/
public StringJoiner add(CharSequence newElement)
//添加一个字符串
```
## String.join()

`String`还提供了一个静态方法`join()`，这个方法在内部使用了`StringJoiner`来拼接字符串，在不需要指定“开头”和“结尾”的时候，用`String.join()`更方便
```java
String[] str = {"小明","小红","小王"};
String s = String.join(",",str);
System.out.println(s);//小明,小红,小王
```




> 知识来源：
[廖雪峰博客](https://www.liaoxuefeng.com/wiki/1252599548343744/1271993169413952)
及 官方api文档