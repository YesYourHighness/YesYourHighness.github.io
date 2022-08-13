---
title: TensorFlow安装
date: 2022-07-17 20:44:04
tags: 
- 机器学习
- TensorFlow
categories: 
- 机器学习


---

<center>
引言：安装TensorFlow
</center>

<!--more-->

# TensorFlow安装

我的系统是Windows10，显卡是NVIDIA GeForce GTX 1060，因此本文会给出本系统的安装教程。

## 参考资料

- NVIDIA[官方文章](https://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows/)

- [TensorFlow安装要求](https://www.tensorflow.org/install/gpu)

## 安装步骤

### 验证显卡是否支持CUDA

首先在[此网站](https://developer.nvidia.com/cuda-gpus)，找到是否有对应显卡的类型

![image-20220718160253972](http://img.yesmylord.cn//image-20220718160253972.png)

### 安装NVIDIA CUDA Toolkit工具

CUDA工具包安装了CUDA驱动程序和创建，构建和运行CUDA应用程序以及库，标题文件和其他资源所需的工具

[下载连接](https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exe_local)

一步一步来，直接安装即可。

![image-20220718163823035](C:\Users\BlackKnight\AppData\Roaming\Typora\typora-user-images\image-20220718163823035.png)

### 下载cuDNN

[下载地址](https://developer.nvidia.com/rdp/cudnn-archive)，选择对应的版本即可，我选择的版本是v8.2

注意，要解压到CUDA的地址里面，比如我这里需要解压到`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.7`

### ANACONDA的使用

这个软件是一个虚拟环境的软件，我们选择TensorFlow的环境，更改一下右方的镜像，这里使用清华大学的镜像

![image-20220718201008470](C:\Users\BlackKnight\AppData\Roaming\Typora\typora-user-images\image-20220718201008470.png)

如图，点击add，添加`http://pypi.tuna.tsinghua.edu.cn/`镜像

安装TensorFlow

```python
pip install tensorflow
```

测试安装是否成功

````python
python -c "import tensorflow as tf;print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
````

![image-20220718200625387](C:\Users\BlackKnight\AppData\Roaming\Typora\typora-user-images\image-20220718200625387.png)

## 遇到报错

安装后，测试安装是否成功遇到：

```
Could not load dynamic library ‘cudnn64_8.dll‘； dlerror: cudnn64_8.dll not found
```

此报错说明你没有下载cuDNN或是下载了没有解压到该有的地方

## 安利一个所见即所得的编辑器——Jupyter

安装

```
pip install jupyter -i https://pypi.tuna.tsinghua.edu.cn/simple
```

Jupyter使用ANACONDA的环境，[此链接](https://developer.nvidia.com/rdp/cudnn-archive)

简化就是两步，安装

```python
pip install ipykernel # 安装
python -m ipykernel install --user --name GV --display-name gv
# 其中GV是自己的环境名，gv是自己想在jupyter中显示的名字
```

