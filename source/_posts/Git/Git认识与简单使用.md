---
title: Git认识与简单使用
date: 2019-07-29 15:54:24
tags:
- Git
categories: 
- Git
---
<center>
引言：

git是什么？为什么要学git？

git上半部分

</center>

<!--more-->

-----------

# Git简介

Git是什么？

世界上最先进的**分布式版本控制系统**。

## 什么是版本控制系统？

记录每次文件的改动，还可以让朋友协作编辑它。

省去自己整理文件的麻烦了

## Git诞生
Linus创建了Linux之后。对于全世界的热心程序员发来的linux代码他和他的团队根本照顾不来。

于是Linus选择了一个商业的版本控制系统BitKeeper，BitKeeper的东家BitMover公司出于人道主义精神，授权Linux社区免费使用这个版本控制系统。

但是，牛人们想破解BitKeeper的协议，被BitMover公司发现了（监控工作做得不错！），于是BitMover公司怒了，要收回Linux社区的免费使用权。

Linus花了两周时间自己**用C**写了一个分布式版本控制系统，这就是Git！

一个月之内，Linux系统的源码已经由Git管理了！

2008年，GitHub网站上线了，它为开源项目免费提供Git存储，无数开源项目开始迁移至GitHub。

## 集中式vs分布式
-  集中式

版本库存放在中央服务器，干活用自己的电脑。需要联网才能工作

好比是一个图书馆，自己借书阅读，填上笔记之后再还回去（当然现实中不能给书加笔记）

- 分布式

版本库在自己的电脑上，工作的时候不需要联网，互相间推送一下，就可以修改啦

# 创建使用版本库

## 创建

创建一个文件夹

进入这个文件夹

输入命令，把这个文件夹变成仓库
```
git init

Initialized empty Git repository in D:/newTemp/.git/
```
此时文件夹下多了一个`.git`文件，又来跟踪管理版本库（这个文件是一个隐藏文件）


## 把文件添加到版本库

注意：

所有的版本控制系统，只能跟踪文本文件的改动，比如TXT文件，网页，所有的程序代码等等

版本控制系统可以告诉你每次的改动，

比如在第5行加了一个单词“Linux”，在第8行删了一个单词“Windows”。

而图片、视频这些二进制文件
，没法跟踪文件的变化，

只能把二进制文件每次改动串起来，只知道图片从100KB改成了120KB，其他无法了解

还有就是不要使用记事本编辑文件，否则可能会出现莫名其妙的Bug

-----

把一个文件放到仓库只需要两步

假设我们有一个`nihao.txt`的文件

他的内容是hello world

1. 把文件放在仓库文件下，输入`git add nihao.txt`
2. 输入`git commit -m "wrote a nihao file"`

```
$ git commit -m "第一次使用git添加了一个文件"
[master (root-commit) 51eb79a] 第一次使用git添加了一个文件
 1 file changed, 1 insertion(+)
 create mode 100644 nihao.txt
```

`-m`后面输入的是本次提交的说明，可以输入本次提交项目的信息

提示 1 file changed,1 insertion（一个文件被改动，插入了一行内容）那么就成功了

可以多次add之后一次性commit

## 查看仓库状态

我们成功的添加并且提交了一个nihao.txt文件

现在我们更改一下文件的内容

添加一行内容bye world

运行`git status`查看仓库的当前状态
```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   nihao.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
输出信息告诉我们

`nihao.txt`已经被修改过了，但是还没有提交修改

使用`git diff`查看修改的内容
```
$ git diff
diff --git a/nihao.txt b/nihao.txt
index 95d09f2..0bf3550 100644
--- a/nihao.txt
+++ b/nihao.txt
@@ -1 +1,2 @@
-hello world
\ No newline at end of file
+hello world
+bye world
\ No newline at end of file
```
通过信息告诉我们，我们添加了一行内容`bye world`

我们再次输入`git add`命令更新文件

然后再次输入`git status`
```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   nihao.txt
```
告诉我们将要提交的修改包括`nihao.txt`

我们可以继续下一步`git commit`了
```
$ git commit -m "修改了文件内容"
[master 8ced12a] 修改了文件内容
 1 file changed, 2 insertions(+), 1 deletion(-)
```
这时候告诉我，一个文件被修改，有两行内容，一个删除（为什么会出现一个删除？？）

我们再再输入一次`git status`
```
$ git status
On branch master
nothing to commit, working tree clean
```
Git告诉我们当前没有需要提交的修改，而且，工作目录是干净（working tree clean）的

## 查看历史记录

使用命令`git log`查看历史记录
```
$ git log
commit 8ced12a07d31eeb810eea3170d337eeab547259e (HEAD -> master)
Author: YesYourHighness <1046467756@qq.com>
Date:   Sun Jul 28 16:38:14 2019 +0800

    修改了文件内容

commit 51eb79a431991cdc8dc1a78f9bab6f3153808a3b
Author: YesYourHighness <1046467756@qq.com>
Date:   Sun Jul 28 16:24:44 2019 +0800

    第一次使用git添加了一个文件

```
我们一共修改了两次文件，显示顺序是从最近到最远

如果嫌输出信息太多，看得眼花缭乱的，可以试试加上`--pretty=oneline`参数：
```
$ git log --pretty=oneline
8ced12a07d31eeb810eea3170d337eeab547259e (HEAD -> master) 修改了文件内容
51eb79a431991cdc8dc1a78f9bab6f3153808a3b 第一次使用git添加了一个文件
```
简单粗暴的显示了一大堆数字字母组合数，它其实是一个十六进制的非常大的数，是有SHA1计算出来的，为了避免大家都用1,2,3...作为版本号导致冲突，所以进行了这样的编排

## 回退版本

首先，git必须知道当前版本是哪一个版本

Git中，当前版本用`HEAD`表示，例如上次输入log命令查询得到的第一条信息，其后有`(HEAD -> master) `

上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个^比较容易数不过来，所以写成`HEAD~100`。

输入`git reset`命令
```
$ git reset --hard HEAD^
HEAD is now at 51eb79a 第一次使用git添加了一个文件
```
`--hard`后面再讲，放心使用

打开文件发现回到了第一个版本的状态，只有一行hello world

输入`git log`我们再看一次当前的版本库的状态

```
$ git log
commit 51eb79a431991cdc8dc1a78f9bab6f3153808a3b (HEAD -> master)
Author: YesYourHighness <1046467756@qq.com>
Date:   Sun Jul 28 16:24:44 2019 +0800

    第一次使用git添加了一个文件
```
只剩下了第一代的版本

可是....

我又后悔了，怎么办

只要你的命令行窗口还没有关掉，往上查找，找到上一个版本的`commit id`

输入`git reset --hard 8ced12`

版本号写前几位即可，git会自动查找

我们惊喜的发现，没错，文件又回来了

Git的版本回退速度非常快，因为Git在内部有个指向当前版本的`HEAD`指针，当你回退版本的时候，Git仅仅是移动了一下指针而已

可是，我把命令行窗口关掉了怎么办！！

输入`git reflog`来查找那次的id，git能然你来回穿梭

## 工作区和暂存区

- 工作区

电脑里能看到的目录，比如说你的仓库的文件夹就是一个目录

但是不包括那个`.git`文件，这是Git的版本库

- 暂存区

git的版本库内有很多东西

最重要的就是stage(或者叫index)的**暂存区**，

还有git为我们自动创建的第一个分支`master`，

以及指向`master`的一个指针`HEAD`

现在我们新建一个文件，放在仓库的目录下

再修改一下`nihao.txt`文件

输入`git status`
```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   nihao.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        dierge.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
清晰的告诉我们，`nihao.txt`被修改了,而`dierge.txt`还没被添加，所以他的状态是`untracked`

使用`git add`添加这两个文件，再用`git status`查看内容
```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   dierge.txt
        modified:   nihao.txt
```
现在，暂存区的状态就变成了这个样子

所以`git add`命令就相当于把所有修改放在了**暂存区**，

然后，

`git commit`把暂存区所有的修改提交到分支

再一次运行`git commit`把改动提交到分支

## 管理修改

现在，我们还是给`nihao.txt`添加一段文字“一次添加”

`git add`再加上`git status`
```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   nihao.txt
```
然后再加上一段文字“二次添加”

然后提交`git commit`
```
$ git commit -m '二次添加会显示吗'
[master 0e6b9f4] 二次添加会显示吗
 1 file changed, 2 insertions(+), 1 deletion(-)
```
提交后再查看状态`git status`,发现第二次的修改没有被提交

用`git diff HEAD -- nihao.txt`命令可以查看工作区和版本库里面最新版本的区别
```
$ git diff HEAD -- nihao.txt
diff --git a/nihao.txt b/nihao.txt
index c514222..e4ab4ba 100644
--- a/nihao.txt
+++ b/nihao.txt
@@ -1,4 +1,5 @@
 hello world
 bye world
 change again
-一次添加
\ No newline at end of file
+一次添加
+二次添加
\ No newline at end of file

```
发现确实没有提交上去

过程：
`第一次修改 -> git add -> 第二次修改 -> git commit`

当我们`git add`时，工作区的第一次修改放入了暂存区，准备提交，但是第二次修改没有进去，所以只是提交了第一次的修改

我们再`git add`和`git commit`提交第二次

## 撤销修改

我们难免犯错误，搞错了一行

我们在`nihao.txt`这个文件加一行 "stupid boss"

### 工作区发现
如果我们发现的及时，在工作区就发现了这个错误

立刻输入`git checkout -- nihao.txt`

输入`git status`发现一切clean

### 暂存区发现
暂存区发现了错误，不要慌张

输入`git reset HEAD nihao.txt`，这个命令可以回退版本，也可以把暂存区的修改撤销

这样就回到了工作区

再次输入`git reset HEAD nihao.txt`

### 版本库发现

如果还没有推到远程版本库，那么赶快版本回退吧

### 远程版本库发现

两个解决办法
- 跑路
- 等死

## 删除文件
现在我们不想要`dierge.txt`这个文件了，怎么删除

本地删除文件`rm dierge.txt`

这时候git已经检测到，你把它删掉了，

输入`git status`检查状态
```
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    dierge.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
提示我们删掉了一个文件

现在有两个选择
1. 删除文件
2. 恢复文件

- 当我们铁了心就是想删除时

输入`git rm dierge.txt`删除文件，并且`git commit`推送
```
$ git commit -m '删除了dierge.txt'
[master fdd980b] 删除了dierge.txt
 1 file changed, 1 deletion(-)
 delete mode 100644 dierge.txt
```

- 当我们删错了的时候

输入 `git checkout -- dierge.txt`

这个命令其实是用版本库里的版本替换工作区的版本，无论是修改还是删除，都可以还原

如果你删掉的文件没有被添加到版本库就被删除，那就真的没有了
