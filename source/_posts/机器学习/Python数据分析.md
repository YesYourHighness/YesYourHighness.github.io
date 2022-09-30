---
title: Python数据分析
date: 2022-09-17 23:44:04
tags: 
- 机器学习
- Python
categories: 
- 机器学习
---

<center>
引言：如何使用Python进行数据分析？属于应用部分
</center>

<!--more-->

# Python数据分析

## 常用的类库

[原文链接](https://github.com/iamseancheney/python_for_data_analysis_2nd_chinese_version/blob/master/%E7%AC%AC01%E7%AB%A0%20%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C.md)

- **numpy** （numerical python）：
  - 提供了多维数组对象ndarray，可以让C等低级语言直接操作，方便快捷（比python原生的数组要快很多）
  - 作为算法和库之间传递数据的容器

- **pandas**（panel data 面板数据）：
  - 此类库常用DataFrame：一个面向列的**二维表结构**
  - 另一个常用Series，一个**一维的标签化数组**对象
  - 兼具 NumPy 高性能的数组计算功能以及电子表格和关系型数据库（如 SQL）灵活的数据处理功能

- matplotlib：
  - 最流行的用于**绘制图表**和其它二维数据可视化的 Python 库
- SciPy：专门解决科学计算中各种标准问题域的包的集合
  - scipy.linalg：扩展了由 numpy.linalg 提供的线性代数例程和矩阵分解功能。
  - scipy.sparse：稀疏矩阵和稀疏线性系统求解器。
- scikit-learn：Python 的通用机器学习工具包
  - 分类算法：SVM、近邻、随机森林、逻辑回归
- statsmodels：
  - 专注于统计分析，而scikit-learn注重预测

命名惯例如下：

```python
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
import statsmodels as sm
```

## numpy

numpy可以快速的处理大数组数据的原因：

1. NumPy 是在一个**连续的内存块**中存储数据，独立于其他 Python 内置对象
2. NumPy 可以在整个数组上执行复杂的计算，而**不需要 Python 的 for 循环**

numpy与python的性能差异对比：

```python
# 创建性能差异对比
%time my_arr = np.arange(100_0000)
%time my_list = list(range(100_0000))
```

```python
# 输出结果
Wall time: 2.01 ms
Wall time: 41.7 ms
```

```python
# 运算时间对比，此处给两个数组都乘2
%time for _ in range(10): my_arr2 = my_arr * 2
%time for _ in range(10): my_list = [x*2 for x in my_list]
```

```python
# 输出结果
Wall time: 29.7 ms
Wall time: 1.39 s
```

### ndarray

> ndarray（N diversion array）ndarray 是一个通用的**同构数据多维容器**

每个数组都有一个 **shape**（一个表示各维度大小的元组）和一个 **dtype**（一个用于说明数组数据类型的对象）

> 创建ndarray











