---
title: Linux有趣的命令
date: 2019-11-14 22:00:33
tags: 
- Linux
categories: 
- 后台
---

<center>
引言：

Linux有趣的小命令
</center>

<!-- more -->

整天对着代码是不是很枯燥，

我找了一晚上这些小玩意儿，把不错的分享一下

闲的发慌就来试试这几个命令吧！！

# Linux有趣的命令

Ubuntu亲试！CentOs把`apt-get`换为`yum`试试，可能不是所有版本通用，

安装前更新一下你的`apt-get`或者`yum`吧
```java
sudo apt-get update
```

## 1.`oneko`

> 一只小猫，追着鼠标跑啊跑，跑啊跑

安装
```
sudo apt-get install oneko
```
运行
```java
oneko //出现一只小猫咪
oneko -dog //出现一只小狗
oneko -sakura //出现一只小樱
oneko -tora //出现一只白条纹老虎，和猫咪很像
```

## 2.`sl`
> 一个火车，注意是sl不是ls

安装
```
sudo apt-get install sl
```
运行
```js
sl //一列火车开过来
sl-h // 没有空格，是一辆长长的火车，CTRL+C退出
```

## 3.`fortune`
> 命运，推荐给你一条名言或者是其他的，最有趣的是竟然有中文版，会推荐给你诗歌、论语、毛主席语录

安装
```
sudo apt-get install fortune //英文版
sudo apt-get install fortune-zh //中文版
```
运行
```
fortune  //即可
```

## 4.`cowsay`
> 牛说，出现一头牛说你让他说的话

安装
```
sudo apt-get install cowsay
```
运行
```java
cowsay I love Linux  //cowsay后面随便加你想让他说的话就好了
fortune|cowsay  //最有意思的是，它能和fortune联动，奶牛就会说出fortune的话

// | 是管道命令符，可以将上一个命令的输出作为下一个命令的输入！学到了！
```

## 5.`cmatrix`
> 黑客帝国，敲上这行命令，就可以装作黑客的样子了！

安装
```
sudo apt-get install cmatrix
```
运行
```
cmatrix
```

## 6.`aafire`
> 一把火烧啊烧啊，一个动态图在你的终端一直烧啊烧啊烧

安装
```
sudo apt-get install libaa-bin
```
运行
```
aafire
```
