---
title: C4.5决策树
date: 2021-06-07 20:00:43
tags: 
- DataMining
- DecisionTree
- C4.5
categories: 
- DataMining
- DecisionTree

---

<center>
引言：

决策树：一种基本的数据挖掘技术

实现决策树的算法有很多种，本篇主要介绍C4.5关于信息熵的算法

</center>

<!--more-->

# C4.5决策树

## 决策树

### 基本概念

> 决策树：一个树状结构。数据挖掘技术的一种，决策树是最常用的分类和预测的技术，可以建立分类和预测模型

### 决策树实现的基本原理

数据挖掘一般都得先从数据源开始，

获得一个有**多个输入属性**和**一个输出属性**的数据集，例如我们这里的数据集：可以看到，有多个输入属性，而唯一的输出属性是“会不会下雨”

| 有没有乌云 | 有没有打雷 | 会不会下雨 |
| ---------- | ---------- | ---------- |
| 有         | 有         | 下雨       |

（这仅仅是一个示例）

然后我们需要对这个数据进行处理，从中选择一个**最**能区别**实例**（可以理解为一行数据，也叫记录、对象等等名称）的属性

然后根据这个属性创建了树的根节点，同时创建这个节点的分支

使用这些分支，将数据集中的数据进行分类，然后重复这个过程，直至完成一棵决策树



我们可以稍微总结一下这个过程：

1. 获得一个“属性-值”的**数据集T**
2. 选择最能区别实例的属性
3. 根据这个属性创建树及其分支
4. 对实例进行分类，进行细分
5. 此时就有了**新的数据集T1**，他是T的子数据集，然后重复2-3步，直至满足：
   - 没有剩余的属性
   - 子类中的实例满足预定义的标准（预定义的标准由自己觉得，可以是全部分到某一个类，也可以是达到一定比例）

### 决策树的三个关键技术

由以上的过程，我们可以看到有三个主要的步骤，这也是就决策树的核心：

- 选择最能区别数据集实例属性的方法
- 剪枝方法
- 检验方法

那么如何判断一个属性是不是最能区别一个实例呢？

这就产生了不同的算法

### 实现算法

每种算法判断一个属性是不是最好的标准不同

1. ID3：基于**信息增益**
2. C4.5：基于**信息增益率**
3. CART：基于**Gini指数**（暂不予讨论）

### 信息熵、信息增益、信息增益率相关公式

#### 信息熵

<img src="http://img.yesmylord.cn//image-20210608095952115.png" alt="image-20210608095952115" style="width:40%;" />

- H(x)表示随机事件x的熵

- p表示xi出现的概率

- xi表示某个随机事件x的所有可能结果

例如：

![image-20210608100153401](http://img.yesmylord.cn//image-20210608100153401.png)

#### 信息增益、信息增益率

​	GainRatio即信息增益率，Gain即信息增益：<img src="http://img.yesmylord.cn//image-20210608100251178.png" alt="image-20210608100251178" style="width:40%;" />

其中分子：Gain

<img src="http://img.yesmylord.cn//image-20210608100412345.png" alt="image-20210608100412345" style="width:40%;" />

<img src="http://img.yesmylord.cn//image-20210608100500873.png" alt="image-20210608100500873" style="width:60%;" />

其中分母：SplitsInfo

<img src="http://img.yesmylord.cn//image-20210608100611824.png" alt="image-20210608100611824" style="width:50%;" />

公式看不懂没关系，看下面这个例子：

### 以一个例子来手动复盘整个过程

根据这个表，使用C4.5算法来构建决策树：

| **序号** | **学历** | **婚姻** | **车** | **收入** | **贷款** |
| -------- | -------- | -------- | ------ | -------- | -------- |
| **1**    | 专科     | 未婚     | 无     | 中       | 否       |
| **2**    | 专科     | 未婚     | 无     | 高       | 否       |
| **3**    | 专科     | 已婚     | 无     | 高       | 是       |
| **4**    | 专科     | 已婚     | 有     | 中       | 是       |
| **5**    | 专科     | 未婚     | 无     | 中       | 否       |
| **6**    | 本科     | 未婚     | 无     | 中       | 否       |
| **7**    | 本科     | 未婚     | 无     | 高       | 否       |
| **8**    | 本科     | 已婚     | 有     | 高       | 是       |
| **9**    | 本科     | 未婚     | 有     | 很高     | 是       |
| **10**   | 本科     | 未婚     | 有     | 很高     | 是       |
| **11**   | 硕士     | 未婚     | 有     | 很高     | 是       |
| **12**   | 硕士     | 未婚     | 有     | 高       | 是       |
| **13**   | 硕士     | 已婚     | 无     | 高       | 是       |
| **14**   | 硕士     | 已婚     | 无     | 很高     | 是       |
| **15**   | 硕士     | 未婚     | 无     | 中       | 否       |

注意：为方便表达，使用log代表log以2为底的计算

1. 首先计算`Info(I)`

观察输出属性“贷款”，共有是、否两种值，是占有9/15，否占有6/15

`Info(I) = -(9/15*log(9/15) + 6/15*log(6/15))`

2. 以属性“学历”为例

观察到学历有三个值：专科、本科、硕士，因此各占5/15

其中:

- 专科：2/5可以贷款
- 本科：3/5可以贷款
- 硕士：4/5可以贷款

于是就如此计算`Info(I, 学历)`

`Info(I, 学历) = 5/15*Info(专科) + 5/15*Info(本科) + 5/15*Info(硕士)`

其中：

`Info(专科) = -(2/5*log(2/5) + 3/5*log(3/5))`

`Info(本科) = -(3/5*log(3/5) + 2/5*log(2/5))`

`Info(硕士) = -(4/5*log(4/5) + 1/5*log(1/5))`

3. 计算`SpiltsInfo(学历)`

`SpiltsInfo(学历) = - (5/15*log(5/15)+ 5/15*log(5/15) + 5/15*log(5/15)) `

4. 计算`Gain(学历)`

这个就很简单了，

`Gain(学历) = Info(I) - Info(I , 学历)`

5. 计算`GainRatio(学历)`

做除法

`GainRatio(学历) = Gain(学历)/SplitsInfo(学历)`

6. 同理依次计算每一个属性的增益率，然后选择增益率最大的为影响最大的属性，以此反复

最后结果：（还真是真实呢！`>_<`）

<img src="http://img.yesmylord.cn//image-20210608103106006.png" alt="image-20210608103106006" style="width:50%;" />

## 决策树代码实现

使用sklearn现有的包tree构建决策树，然后再使用graphvz绘制

注意：

1. 需要先下载grapVz并配置环境变量
2. 把数据保存到`xueli.txt`文件中，然后进行读取
3. 核心是`DecisionTreeClassifier`这个方法

```python
# -*- coding: utf-8 -*-
from io import StringIO

import pandas
import pydotplus
from sklearn import tree
from sklearn.preprocessing import LabelEncoder
from sklearn.tree import DecisionTreeClassifier

def createDataSet():
    with open('xueli.txt', 'r', encoding='utf-8') as fr:  # 加载文件
        dataSet = [inst.strip().split(" ") for inst in fr.readlines()]  # 处理文件
    labels = "xueli hunyin che shouru".split(" ") # 属性，最后是一个列表，注意！不要写入输出属性
    return dataSet, labels

if __name__ == '__main__':
    dataSet, labels = createDataSet()
    yDataList = []  # 提取每组数据的类别，保存在列表里
    for each in dataSet:
        yDataList.append(each[-1])

    dataDict = {}
    for each_label in labels:
        tempList = list()
        for each in dataSet:
            tempList.append(each[labels.index(each_label)])
        dataDict[each_label] = tempList

    dataPD = pandas.DataFrame(dataDict)

    leDict = dict()
    for col in dataPD.columns:
        leDict[col] = LabelEncoder()
        dataPD[col] = leDict[col].fit_transform(dataPD[col])

    dt = DecisionTreeClassifier(criterion='entropy')  # entropy代表使用信息熵
    dt.fit(dataPD.values.tolist(), yDataList)

    dot_data = StringIO()
    tree.export_graphviz(dt, out_file=dot_data,  # 绘制决策树
                         feature_names=dataPD.keys(),
                         class_names=dt.classes_,
                         filled=True, rounded=True,
                         special_characters=True)
    graph = pydotplus.graph_from_dot_data(dot_data.getvalue())
    graph.write_png("tree.png")
```

