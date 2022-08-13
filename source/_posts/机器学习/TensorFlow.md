---
title: TensorFlow入门
date: 2022-07-19 20:44:04
tags: 
- 机器学习
- TensorFlow
categories: 
- 机器学习



---

<center>
引言：什么是Tensor？基本入门
</center>

<!--more-->

# TensorFlow入门

## 什么是TensorFlow

TensorFlow是机器学习最流行的框架之一

## Tensor

### 什么是张量？

[Tensor](https://www.tensorflow.org/guide/tensor)：**张量**，有很多种[意义](https://www.zhihu.com/question/20695804/answer/447498656)，但是在机器学习中，**张量是向量和矩阵到可能更高维度的推广**（原文如下，根本不理解什么意思好吧）

> A tensor is a generalization of vectors and matrices to potentially higher dimensions. Internally, TensorFlow represents tensors as n-dimensional arrays of base datatypes.

在TensorFlow内，`dType`（里面具有很多类型，包括`int`等等）构成的数组就是张量（张量是具有**统一类型**的**多维数组**）

一句话概括一下：**在计算机的范畴内，张量就是具有统一类型的多维数组**

### 创建张量

创建一个张量只需要给出两个内容：值与类型（类型不指定，将使用默认的类型）

- 0维的张量，就是一个数字，或者称为 **标量**，或称为 **0秩张量**

```python
rank_0_tensor = tf.constant(4) # 4就是一个标量，也叫一个0维张量
# constant() 会创建一个常量
print(rank_0_tensor) 
# 输出 tf.Tensor(4, shape=(), dtype=int32)
```

- 一维数组，就是一维的张量，**1秩张量**，有一个轴

```python
rank_1_tensor = tf.constant([2.0, 3.0, 4.0])
print(rank_1_tensor) 
# 输出 tf.Tensor([2. 3. 4.], shape=(3,), dtype=float32)
```

- 二维数组就是二维张量，**2秩张量**，对于更高维度将不再赘述

注意：**维（Dimension）**和**秩（Rank / Degree）**含义不同。可能会看到“二维张量”之类的表述，但 2 秩张量通常并不是用来描述二维空间。

可以使用`ones()`、`zeros()`来创建张量

```python
test1 = tf.ones(3) # 创建1秩tensor，并用1填充
test2 = tf.ones([2,2]) # 创建2秩tensor，并用1填充
test3 = tf.ones([2,2,2]) # 创建3秩tensor，并用1填充
print(test1)
print(test2)
print(test3)
```

输出内容如下，可以看到，如果参数是一个数组，那么这个数组表示这个张量的**形状**

```
tf.Tensor([1. 1. 1.], shape=(3,), dtype=float32)

tf.Tensor(
[[1. 1.]
 [1. 1.]], shape=(2, 2), dtype=float32)
 
tf.Tensor(
[[[1. 1.]
  [1. 1.]]

 [[1. 1.]
  [1. 1.]]], shape=(2, 2, 2), dtype=float32)
```

### 张量的属性

- **type 类型**：元素的类型是什么（比如int16、int32、string）
- **shape 形状**：每个轴的元素个数（或者说每个轴的长度）
- **rank/degree 秩**：有多少个轴
- **dimension 维**：某个特定的轴
- **elements 元素**：张量含有元素的多少，是各个形状的乘积

### 张量的种类

张量有不同的种类

- Variable
- Constant
- Placeholder
- SparseTensor

只有`Variable`是可变的张量，其他张量都是不可变的

```python
tf.Variable(x1,x2) # x1表示内容，x2表示类型
tf.constant(x1,x2) #同上
```

### 改变张量的形状

`reshape`函数可以更改张量的形状，但前提是：**重构张量中的元素数量必须与原始张量中的元素数量相匹配**

```python
tensor1 = tf.ones([1,2,3])
tensor2 = tf.reshape(tensor1, [2,3,1])
tensor3 = tf.reshape(tensor2, [-1]) # -1 会自动算出该维度的数量 ，这样就是一个 6个长度的一维数组
# 如果选择 tf.reshape(tensor2, [3,-1]) 这样就是一个 3*2 的二维数组
```

输出结果为

```python
tf.Tensor(
[[[1. 1. 1.]
  [1. 1. 1.]]], shape=(1, 2, 3), dtype=float32)
tf.Tensor(
[[[1.]
  [1.]
  [1.]]

 [[1.]
  [1.]
  [1.]]], shape=(2, 3, 1), dtype=float32)
tf.Tensor([1. 1. 1. 1. 1. 1.], shape=(6,), dtype=float32)
```

### 对张量进行切片

`tensor[dim1, dim2, dim3...]`每个位置上对应的数字就找到了对应的元素

用一个例子来介绍切片（类似于py的切片）

```python
matrix = [[1,2,3,4,5],
          [6,7,8,9,10],
          [11,12,13,14,15],
          [16,17,18,19,20]]

tensor = tf.Variable(matrix, dtype=tf.int32) 
print(tf.rank(tensor)) # tf.Tensor(2, shape=(), dtype=int32)
print(tensor.shape) # (4, 5)
```

切片操作：

```python
three = tensor[0,2]  # 找到第一行，第三列的元素 3

row1 = tensor[0]  # 选择了第一行

column1 = tensor[:,0]  # 选择了第一列

row_2_and_4 = tensor[1::2]  # 第二行与第三行

column_1_in_row_2_and_3 = tensor[1:3, 0] # 选择行[1,3)，每行的第1个元素
```

结果如下：

```
tf.Tensor(3, shape=(), dtype=int32)
tf.Tensor([1 2 3 4 5], shape=(5,), dtype=int32)
tf.Tensor([ 1  6 11 16], shape=(4,), dtype=int32)
tf.Tensor(
[[ 6  7  8  9 10]
 [16 17 18 19 20]], shape=(2, 5), dtype=int32)
tf.Tensor([ 6 11], shape=(2,), dtype=int32)
```

`tensor[:,0]`为什么这行代码选择了第一列？

答：第一个数`:`表示选择所有的行，而第二个0表示选择每行的第一个数字，因此是第一列

[关于切片的操作，可以看此篇blog](https://www.yesmylord.cn/2021/05/01/Python/Python%E5%BA%8F%E5%88%97/#%E5%88%97%E8%A1%A8%E6%94%AF%E6%8C%81%E5%88%87%E7%89%87%E6%93%8D%E4%BD%9C)

### Tensor的基本API总结

创建

```python
tf.Variable(x1,x2) # 创建变量
tf.constant(x1,x2) # 创建常量
tf.ones(x) # x为一个数或一个数组，创建如数组所示形状的张量，并用1填充
tf.zeros(x) # 同上，使用0填充
```

查看属性

```python
tf.rank(x) # 查看x张量的秩
tensor1.shape # 查看张量的形状（注意这个并不是方法，而是一个属性）
```

更改形状

```python
tf.reshape(tensor1, x2) # x2为形状的数组，更改前后元素数必须相同，-1可以自动计算数量
```

关于切片的操作，可以看[此篇blog](https://www.yesmylord.cn/2021/05/01/Python/Python%E5%BA%8F%E5%88%97/#%E5%88%97%E8%A1%A8%E6%94%AF%E6%8C%81%E5%88%87%E7%89%87%E6%93%8D%E4%BD%9C)

## 参考资料

- [Tensorflow官方文档](https://www.tensorflow.org/guide/tensor)
- [TensorFlow基本介绍](https://colab.research.google.com/drive/1F_EWVKa8rbMXi3_fG0w7AtcscFq7Hi7B#forceEdit=true&sandboxMode=true)
- [机器学习核心算法](https://colab.research.google.com/drive/15Cyy2H7nT40sGR7TBN5wBvgTd57mVKay#forceEdit=true&sandboxMode=true&scrollTo=tUgsCvCHLksw)



