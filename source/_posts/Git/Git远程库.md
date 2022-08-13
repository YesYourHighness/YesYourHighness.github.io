---
title: Git远程库
date: 2019-07-29 15:54:37
tags:
- Git
categories: 
- Git
---
<center>
引言：

git的远程仓库如何建立？

git下半部分
</center>

<!-- more -->

----

# 远程仓库（远程版本库）

Git是分布式版本控制系统，同一个Git仓库，可以分布到不同的机器上。

怎么分布呢？最早，肯定只有一台机器有一个原始版本库，此后，别的机器可以“克隆”这个原始版本库，而且每台机器的版本库其实都是一样的，并没有主次之分。

实际情况通常是这样：

找一台电脑充当服务器的角色，每天24小时开机，
其他每个人都从这个“服务器”仓库克隆一份到自己的电脑上，
并且各自把各自的提交推送到服务器仓库里，也从服务器仓库中拉取别人的提交。

注册一个GitHub账号，就可以免费获得Git远程仓库。

由于你的本地Git仓库和GitHub仓库之间的传输是通过SSH加密的，所以，需要一点设置：

1. 创建SSH Key

在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：
```
ssh-keygen -t rsa -C "youremail@example.com"
```
注意这里的`-C`是大写的
，然后一路enter，使用默认值

之后我们可以在 **用户主目录(~)** 找到一个`.ssh`的文件目录，内部有`id_rsa`和`id_rsa.pub`，这两个就是SSH Key的密钥对，前者是私钥，不能泄露出来，后者是公钥，可以告诉任何人

2. 登录Github

点击步骤：

`setting -> SSH and GPG keys ->  New SSH -> 随便输入一个Title -> 把id_rsa.pub文件的内容复制到key文本框内`

为什么GitHub需要SSH Key呢？

因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。

最后友情提示，在GitHub上免费托管的Git仓库，任何人都可以看到喔（但只有你自己才能改）。

如果你不想让别人看到Git库，有两个办法

一个是交点保护费，让GitHub把公开的仓库变成私有的，这样别人就看不见了（不可读更不可写）。

另一个办法是自己动手，搭一个Git服务器，因为是你自己的Git服务器，所以别人也是看不见的。这个方法我们后面会讲到的，相当简单，公司内部开发必备。

## 添加远程库

我们还得再github上建一个库

点击步骤：

`new repository -> 输入库的名称 -> 其他默认，点击Create repository`

目前，这个库是空的

我们可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联

现在我们在本地的仓库下运行
```
$ git remote add origin git@github.com:YourGitHubId/learngit.git
```
添加后，远程库的名字就是`origin`，这是git默认的叫法

输入`git push -u origin master`
就可以把本地库的所有内容推送到远程库

使用这个命令，其实是吧当前分支`master`推送到了远程

由于远程库是空的，我们加了`-u`参数，Git不但会把本地的`master`分支内容推送到远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，以后推送或者拉取时就可以简化命令

注意：执行这条命令，你的**版本库**必须有东西

如果你的版本库是空的，则会报错
```
error: src refspec master does not match any
```

成功后我们就可以在github上看到远程库的内容和本地的一模一样

从此，只要本地做了提交，就可以通过命令
```
git push origin master
```
把本地`master`分支的最新修改推送至GitHub

### SSH警告
当第一次使用Git的`clone`或者`push`命令连接到Github上时，会得到一个警告
```
The authenticity of host 'github.com (xx.xx.xx.xx)' can't be established.
RSA key fingerprint is xx.xx.xx.xx.xx.
Are you sure you want to continue connecting (yes/no)?
```
这是因为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要你确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入yes回车即可。

Git会输出一个警告，告诉你已经把GitHub的Key添加到本机的一个信任列表里了：
```
Warning: Permanently added 'github.com' (RSA) to the list of known hosts.
```
这个警告只会出现一次，后面的操作就不会有任何警告了。

## 从远程仓库克隆

现在，假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆。

登录GitHub，建立一个新的仓库

勾选`Initialize this repository with a README`,这样GitHub会自动给我们创建一个`README.md`的文件

使用命令
```
$ git clone git@github.com:YourGitHubId/gitskills.git
```
就可以克隆到你当前的位置了


你也许还注意到，GitHub给出的地址不止一个，还可以用`https://github.com/YourId/gitskills.git`

这样的地址。实际上，Git支持多种协议，默认的`git://`使用ssh，但也可以使用https等其他协议。

使用https除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用ssh协议而只能用https。

# 分支管理

分支有什么用呢？

创建了一个属于你自己的分支，别人看不到，还继续在原来的分支上正常工作，而你在自己的分支上干活，想提交就提交，直到开发完毕后，再一次性合并到原来的分支上。

## 创建与合并分支

`HEAD`指向当前分支, `master`指向提交


- 在最开始

`matser`是一条线,Git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支，以及当前分支的提交点
![image](https://static.liaoxuefeng.com/files/attachments/919022325462368/0)
- 每次提交

`master`分支都会向前移动一步，随着不断的提交，`master`分支也就会越来越长

- 当我们创建新的分支，例如`dev`

Git创建了一个新的指针`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上
![image](https://static.liaoxuefeng.com/files/attachments/919022363210080/0)

由于是指针，所以Git创建分支非常快

从现在开始，对工作区的修改和提交就是对`dev`分支,比如新提交一次后，dev指针往前移动一步，而master指针不变：
![image](https://static.liaoxuefeng.com/files/attachments/919022387118368/0)

假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上，直接把`master`指向`dev`的当前提交，就完成了合并
![image](https://static.liaoxuefeng.com/files/attachments/919022412005504/0)

合并完成后，可以删除`dev`分支，删除`dev`分支就是把`dev`指针给删掉，删掉后我们就剩下了一条`masrer`分支
![image](https://static.liaoxuefeng.com/files/attachments/919022479428512/0)

知道原理我们开始实战：

第一步：

新建一个文件夹->`git init`->新建一个文件，随便加上些内容->`git add`->`git commit`

第二步：

创建一个分支,输入命令
```
 git branch dev
```
第三步：

切换到该分支
```
git checkout dev
```
```
BlackKnight@LAPTOP-JQFP1S4M MINGW64 /d/gitLearn (master)
$ git branch dev

BlackKnight@LAPTOP-JQFP1S4M MINGW64 /d/gitLearn (master)
$ git checkout dev
Switched to branch 'dev'

BlackKnight@LAPTOP-JQFP1S4M MINGW64 /d/gitLearn (dev)

```
(
第二步第三部可以合为一步：
```
git checkout -b dev
```
创建并切换一气呵成
)

第四步：

输入命令

```
git branch
```
查看当前分支
```
$ git branch
* dev
  master
```
处于哪一个分支，前面会有一个*号

第五步：

修改文件内容,并且`git add`执行`git commit`
，然后切换回`master`分支
，查看文件，发现修改的内容不见了

第六步：

合并`dev`分支的内容到`master`上
,输入命令

```
git merge dev
```
`git merge`命令用于合并指定分支到当前分支

再次查看文件，发现修改回来了
```
$ git merge dev
Updating 7666beb..19ed9b4
Fast-forward
 hello.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
```
注意到上面有`Fast-forward`信息，意思是“快进模式”，也就是会直接把`master`指向`dev`的当前提交，所以合并速度十分快，之后会将其他的合并方式

第七步：

合并完成后，可以删除`dev`分支了
```
git branch -d dev
```
输入`git branch`查看分支状态
```
$ git branch
* master
```
发现只有一个分支了

## 解决冲突

合并分支，可能遇到冲突的问题

准备一个新的分支
```
$ git checkout -b featurel
Switched to a new branch 'featurel'
```

修改文件的内容,最后一行加一个bye world,并且添加`add`提交`commit`

切换到`master`分支`git checkout master`

更改文件内容,最后一行加`hello world`,然后继续`add`和`commit`

现在，分支就变成了这个样子
![image](https://static.liaoxuefeng.com/files/attachments/919023000423040/0)

合并分支
```
$ git merge featurel
Auto-merging hello.txt
CONFLICT (content): Merge conflict in hello.txt
Automatic merge failed; fix conflicts and then commit the result.
```
消息告诉我们，文件存在冲突，必须手动解决冲突再提交

打开我们的文件
```
<<<<<<< HEAD
hello world
=======
bye world
>>>>>>> featurel
```
git用这种方式标记出不同分支的内容

我们修改为bye world后再`add`提交`commit`

现在分支成为了这个样子
![image](https://static.liaoxuefeng.com/files/attachments/919023031831104/0)

用`git log --graph`可显示分支图
```
*   commit 24143f569703452d66dd7c4df3c65ed48a70aec5
|\  Merge: 6b29d3c 4c0b8fa
| | Author: YesYourHighness <1046467756@qq.com>
| | Date:   Mon Jul 29 11:05:23 2019 +0800
| |
| |     conflict fixed
| |
| * commit 4c0b8fa27ff654311802bc3db98dc085cd4e0dd3 (featurel)
| | Author: YesYourHighness <1046467756@qq.com>
| | Date:   Mon Jul 29 10:54:24 2019 +0800
| |
| |     init
| |
* | commit 6b29d3cabf64b2ac9a65cc2677dbcb41b8bc4cfc
|/  Author: YesYourHighness <1046467756@qq.com>
|   Date:   Mon Jul 29 10:55:40 2019 +0800
|
```
最后删除分支

## 分支管理策略

合并分支时，Git会尽量用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

我们实战一下`--no-ff`方式的`git merge`

首先创建并切换`dev`分支
```
git checkout -b dev
```
修改文件内容，并且提交一个新的commit

然后切换回`master`

然后合并分支，
注意`--no-ff`参数表示禁用`Fast forward`模式
,因为会自动创建一个commit我们需要加`-m`
```
$ git merge --no-ff -m 'merge with no-ff' dev
Merge made by the 'recursive' strategy.
 hello.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
```
我们用`git log --graph`查看分支
```
*   commit 061646bf6b4321746b6f27f1926a528f7dec16e3 (HEAD -> master)
|\  Merge: bb2ba62 405dd2a
| | Author: YesYourHighness <1046467756@qq.com>
| | Date:   Mon Jul 29 11:19:31 2019 +0800
| |
| |     merge with no-ff
| |
| * commit 405dd2a7fc7c047c837eee1fac4976a4c213e555 (dev)
|/  Author: YesYourHighness <1046467756@qq.com>
|   Date:   Mon Jul 29 11:17:29 2019 +0800
|
|       change in dev
|
```

### 策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以

## Bug分支

假设你接到一个修复一个代号101的bug的任务时，很自然地，你想创建一个分支issue-101来修复它，但是你当前正在dev上进行到一半的工作还没有提交

Git还提供了一个stash功能，
可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：
```
$ git stash
Saved working directory and index state WIP on master: 061646b merge with no-ff
```
用`git status`查看发现是干净的(除非有没有被Git管理的文件)

首先确定要在哪个分支上修复bug，假定需要在master分支上修复，就从master创建临时分支

修复完成后，切换到master分支，并完成合并，最后删除分支

接着回去`dev`
```
git checkout dev
```
发现工作区是干净的,输入
```
$ git stash list
stash@{0}: WIP on master: 061646b merge with no-ff
```
工作现场还在，只是保存到另一个地方了

需要恢复有两个办法

第一种：

`git stash apply`恢复后，stash内容并不删除，需要`git stash drop`来删除

第二种：

`git stash pop`恢复的同时删除

这是再查`git stash list`就看不到任何内容了

你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令：
```
$ git stash apply stash@{0}
```

## Feature分支

软件开发中，总有新的功能要不断添加进来。

添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

于是准备开发，在当你`add`并且提交新的分支之后，切回主分支

但是经理要求阉割此功能，于是我们要删除这个功能

```
$ git branch -d wq
error: The branch 'wq' is not fully merged.
If you are sure you want to delete it, run 'git branch -D wq'.
```
git阻止了操作，提醒你删除后将丢失数据，强行删除用大写的D

现在我们强行删除
```
$ git branch -D wq
Deleted branch wq (was e055832).
```
删除成功

## 多人协作

当你从远程仓库克隆时，实际上Git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名称是`origin`。

要查看远程库的信息
```
git remote
```
加v可以获得更详细的信息
```
git remote -v
```
这条命令显示了可以抓取和推送的origin的地址。如果没有推送权限，就看不到push的地址。

推送分支，就是把该分支上的所有本地提交推送到远程库
```
git push origin master
```
如果要推送其他分支，比如`dev`，就改为
```
git push origin dev
```
- `master`分支是主分支，因此要时刻与远程同步；

- `dev`分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；

- `bug`分支只用于在本地修复`bug`，就没必要推到远程了，除非老板要看看你每周到底修复了几个`bug`；

- `feature`分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

抓取分支

当你从远程库clone时，默认情况下，你只能看到本地的master分支

所以想要在`dev`上开发就必须创建远程的`origin`的`dev`分支到本地
```
git checkout -b dev origin/dev
```
现在就可以在`dev`上修改推送了

多人推送，难免会发生冲突

解决办法，先把最新的提交`pull`下来
```
$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> dev
```
在本地合并解决冲突在推送,但是我们需要先建立本地`dev`与远程`origin/dev`的链接
```
$ git branch --set-upstream-to=origin/dev dev
Branch 'dev' set up to track remote branch 'dev' from 'origin'.
```
解决冲突，再推就可以了

因此，多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；

2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；

3. 如果合并有冲突，则解决冲突，并在本地提交；

4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

# 标签

## 创建标签
切换到需要标签的分支
```
$ git tag v1.0
```
查看所有标签
```
git tag
```
也可以找到`commit id`来打标签
```
 git tag v0.9 f52c633
```
标签不是按时间顺序列出，而是按字母排序的。

还可以创建带有说明的标签,`-a`指定标签名`-m`指定说明文字
```
 git tag -a v0.1 -m "version 0.1 released" 1094adb
```

可以用`git show <tagname>`查看标签信息：
```
$ git show v1.0
commit 061646bf6b4321746b6f27f1926a528f7dec16e3 (HEAD -> master, tag: v1.0)
Merge: bb2ba62 405dd2a
Author: YesYourHighness <1046467756@qq.com>
Date:   Mon Jul 29 11:19:31 2019 +0800

    merge with no-ff
```

如果标签打错了，也可以删除：
```
$ git tag -d v0.1
Deleted tag 'v0.1' (was f15b0dd)
```
因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除

如果要推送某个标签到远程，使用命令git push origin <tagname>：
```
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
 * [new tag]         v1.0 -> v1.0
```
或者，一次性推送全部尚未推送到远程的本地标签：
```
$ git push origin --tags
Total 0 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
 * [new tag]         v0.9 -> v0.9
 ```
 如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：
```
$ git tag -d v0.9
Deleted tag 'v0.9' (was f52c633)
```
然后，从远程删除。删除命令也是push，但是格式如下：
```
$ git push origin :refs/tags/v0.9
To github.com:michaelliao/learngit.git
 - [deleted]         v0.9
```