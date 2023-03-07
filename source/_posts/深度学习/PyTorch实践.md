---
title: 卷积神经网络CNN
date: 2022-10-15 18:11:23
tags: 
- PyTorch
- 神经网络
categories: 
- 深度学习
---

<center>
引言：卷积神经网络 CNN
</center>


<!--more-->

相关资料：

- b站 [刘二大人](https://www.bilibili.com/video/BV1Y7411d7Ys/?from=search&seid=1631997590037031874&spm_id_from=333.337.0.0&vd_source=c8709f8826bf296abab8aeee72b0e338)

# 基本的CNN

## 数据准备

以MINST数据为例，搭建CNN并进行测试。

MINST：入门级的CV数据库，内容全为手写的阿拉伯数字，包含了6w张训练集图片+1w张测试集图片

```python
import torch
from torchvision import transforms # 数据原始处理
from torchvision import datasets # datasets包含了MINST数据集
from torch.utils.data import DataLoader # 分批量载入数据
import torch.nn.functional as F # 激活函数
import torch.optim as optim # 优化器
```

### 梯度下降存在的问题

[梯度下降 知乎Link](https://zhuanlan.zhihu.com/p/113714840)

- **Batch梯度下降** BGD——遍历所有数据，计算损失函数，计算梯度，更新梯度

计算量大，收敛速度慢，训练的模型一般

- **随机梯度下降** SGD ——每次从训练集中随机选择一个样本，计算其对应的损失和梯度，进行参数更新，反复迭代

收敛速度比batch还要慢，还遇到**鞍点问题**；但是训练的模型好。

> **鞍点saddle point 问题**：目标函数在此点的**梯度为0**，但从该点出发的一个方向存在函数极大值点，而另一个方向是函数的极小值点，在高度非凸空间中，存在大量的鞍点，这使得梯度下降法有时会失灵，虽然不是极小值，但是看起来确是收敛的。 

> 注意：**鞍点和局部最优解 local minima不同**

critical point：包括local minima局部最优解 和 saddle point鞍点

![鞍点与局部最优解](http://img.yesmylord.cn//image-20221016165411619.png)

> 如何鉴别是鞍点还是局部最优点？
>
> 是否可以**逃离**：可以逃离的是鞍点，逃不掉的是局部最优解
>
> 这部分可以看这个视频[bilibili link](https://www.bilibili.com/video/BV1zP4y1p73V/?spm_id_from=333.337.search-card.all.click&vd_source=c8709f8826bf296abab8aeee72b0e338)：
>
> 根据hessian判断
>
> - Hessian的所有特征值均大于0：local minima
> - Hessian的所有特征值均小于0： max minima
> - Hessian的特征值有时候大于0，有时候小于0：saddle point

- 综合两者DataLoader——MINI-Batch

使用MINI-Batch时要使用一个嵌套的循环：

```python
for epoch in range(train_epochs):
    for i in range(total_batch):
        # 每次执行一个MINI-Batch
```

> 三个重要的概念：：
>
> - **Epoch**：**所有样本**都经过一个**前馈+反馈**，就叫一个Epoch
> - **Batch-size**：一个前向+反向所用的样本的数量
> - **Iteration**：前+反的次数

显然有：`Epoch = Batch-size * Iteration`

### 载入数据的过程

在DataSet准备好样本数据后，会经过一个Shuffle过程，将样本数据打乱

在这之后，在进行分批，每一批就是一个Batch

![数据载入过程](http://img.yesmylord.cn//image-20221015182258732.png)



## 数据预处理

这里关于`transform`做一个API介绍：[这一部分的官方Link](https://pytorch.org/vision/stable/transforms.html#transforms-on-pil-image-and-torch-tensor)

- `Compose()`：将多个转换操作组合在一起

- `ToTensor()`：将一个pillow图片或是ndarray转换为一个张量（即将0-255转为0-1）
- `Normalize`：用**均值 mean**和**标准差 std**归一化一个浮点张量图像（这个操作只能操作torch的tensor）
  - 参数`mean`：每个通道的均值序列
  - 参数`std`：每个通道的标准差序列
  - 计算公式：`image = (image - mean)/std`

```python
batch_size = 64
# 前部分：将数据从原本的pillow（python处理图片的库）格式转为tensor，即将0-255转为0-1
# 后半部分：将每个channel的数据转换为标准高斯分布，两个参数：均值mean、方差std；计算式：
transform = transforms.Compose([transforms.ToTensor(), transforms.Normalize((0.1307,), (0.3081,))])
# 这里的MIST数据集就一个通道因此只需要传一个值的序列

# 没有数据会自动下载
train_dataset = datasets.MNIST(root='../dataset/mnist/', train=True, download=True, transform=transform)
train_loader = DataLoader(train_dataset, shuffle=True, batch_size=batch_size)
test_dataset = datasets.MNIST(root='../dataset/mnist/', train=False, download=True, transform=transform)
test_loader = DataLoader(test_dataset, shuffle=False, batch_size=batch_size)
```

## 神经网络模型设计

[torch.nn官方API：](https://pytorch.org/docs/stable/generated/torch.nn.Module.html#module)，这里列几个用到的：

- `torch.nn.Module`：所有神经网络的类的**基类**
- `torch.nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0...)`：二维卷积
  - 参数1`in_channels`：输入通道
  - 参数2`out_channels`：输出通道
  - 参数3`kernel_size`：卷积核的大小
  - 参数4`stride`：步长
  - 参数5`padding`：图形填充0
- `torch.nn.MaxPool2d(kernel_size)`：二维最大值池化
  - 参数`kernel_size`：核大小
- `torch.nn.Linear`：linear unit线性单元`y = xA + b`
  - 参数1`in_channels`：输入通道
  - 参数2`out_channels`：输出通道
  - 参数3`bias`：默认为True

- `tensor.view()`：用来改变tensor的形状，数据不会变
  - 返回的新tensor与原tensor**共享内存**，意味着你改变一个，另一个也会改变
  - 当某一维度是`-1`，会自动计算这一维度的大小

PyTorch固定的模板就是这样：`__init__`初始化卷积核、池化、全连接层，`forward`写神经网络的执行顺序，反馈由Module自动执行无需我们设计

```python
# 神经网络模型
class Net(torch.nn.Module): # 继承nn.Module
    def __init__(self):
        super(Net, self).__init__()
        # Conv2d参数：输入通道、输出通道、核的大小
        self.conv1 = torch.nn.Conv2d(1, 10, kernel_size=5)
        self.conv2 = torch.nn.Conv2d(10, 20, kernel_size=5)
        # 池化2*2
        self.pooling = torch.nn.MaxPool2d(2)
        # 全连接：输入通道，输出通道，结果为0-9数字
        self.fc = torch.nn.Linear(320, 10)
        
    def forward(self, x):
        # flatten data from (n,1,28,28) to (n, 784)
        batch_size = x.size(0) # 0是x大小第1个参数，自动获取batch大小
        # 卷积核1->池化->激活->卷积核2->池化->激活
        x = F.relu(self.pooling(self.conv1(x)))
        x = F.relu(self.pooling(self.conv2(x)))
        x = x.view(batch_size, -1) # -1 此处自动算出的是320
        # 全连接，拉成一维
        x = self.fc(x)
        return x
# 实例化
model = Net()
device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")# 如果有GPU使用gpu
model.to(device) # 将model存到显卡
```

## 损失与优化

```python
# 损失函数：该准则计算输入和目标之间的交叉熵损失
criterion = torch.nn.CrossEntropyLoss()
# 优化器：负责训练模型，反向传播更新参数：lr是学习率
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.5)
```

### 损失函数

> 损失函数也是Module的子类，他负责计算损失，计算损失的算法有很多种

- `MSELoss`：求y与y_hat的差值平方
- `CrossEntropyLoss`：交叉熵

> **交叉熵**：它主要刻画的是**实际输出（概率）与期望输出（概率）的距离**，也就是交叉熵的值越小，两个概率分布就越接近

### 优化器

> [优化器](https://pytorch.org/docs/stable/optim.html)：optim是一个实现了多个优化算法的包，直接调用即可。

构造一个优化器：使用前需要创建一个优化器对象，必须传入一个module的参数对象，并可以配置学习率lr

- `model.parameters()`：检查model所有单元的参数，发现有权重就拿走（假如内部有一个线性单元linear unit，就会调用他的`linear.parameters()`方法，如果还有其他单元，回依次获取）

使用：只要梯度更新（例如`backward()`），就可以调用优化器方法`step()`

## 训练

整个训练过程可以精炼为如下几个步骤：

1. 计算`y_hat`
2. 计算损失`loss`
3. 反馈`backward()`（**注意**：在此之前记得优化器清零`zero_grad()`）
4. 更新`step()`

````python
# 训练及更新
def train(epoch):
    running_loss = 0.0
    for batch_idx, data in enumerate(train_loader, 0):# 指定从零开始
        inputs, target = data
        # 有GPU使用GPU
        inputs, target = inputs.to(device), target.to(device)  
        # 计算y_hat
        outputs = model(inputs)
        # 前馈：计算损失
        loss = criterion(outputs, target)
        # 优化器清零：backward是累积的，因此在backward前记得将所有权重清0
        optimizer.zero_grad()
        # 反馈：计算梯度
        loss.backward()
        # 更新：根据梯度和学习率更新
        optimizer.step()
        # 求和损失
        running_loss += loss.item()
        # 300次计算一次平均损失
        if batch_idx % 300 == 299:
            print('[%d, %5d] loss: %.3f' % (epoch+1, batch_idx+1, running_loss/300))
            running_loss = 0.0
````

## 预测

```python
def test():
    correct = 0
    total = 0
    with torch.no_grad(): # 无需计算梯度
        for data in test_loader:
            images, labels = data
            images, labels = images.to(device), labels.to(device)
            # 用训练的模型测试images
            outputs = model(images)
            _, predicted = torch.max(outputs.data, dim=1)
            total += labels.size(0)
            # 计算正确率
            correct += (predicted == labels).sum().item()
    print('accuracy on test set: %d %% ' % (100*correct/total))
```

# 高级的CNN

## Inception

GoogleNet是一个比较出名的CNN模型

> Inception：对于比较复杂的结构，会有很多复用的结构，这样的结构就叫做Inception

比如这样的一个Inception结构，优点是：往往并不能确定什么样的卷积核比较好，因此一个Inception就可以带多个卷积核，来比较他们之间的性能，如果其中一条线路的效果较好，那么就增大这条线的权重

![Inception](http://img.yesmylord.cn//image-20221015190716282.png)

Concatnate会将不同通道的tensor合并在一起

### 1*1的卷积核

> `1*1`卷积核的作用：**信息融合，以便于降低运算量**
>
> 在多通道的情况下，会将每个卷积核的特征融合到一起
>
> （类比与学校的加权成绩）

![1*1卷积核的作用](http://img.yesmylord.cn//image-20221015192557769.png)

降低运算量：假设现在有要对`192*28*28`的张量进行卷积，卷积核为`5*5`，最后输出`32*28*28`：

那么一共要`5*5*32*192*28*28=120_422_400`

如果给中间加一步`16`个卷积`1*1`，那么运算量会变为`1*1*28*28*192*16 + 5*5*28*28*16*32=12_433_648`

可见，降低了1/10的运算量

### 代码实现

![Inception](http://img.yesmylord.cn//image-20221015190716282.png)

设这四条线路分别为A、B、C、D，我们在构建模型时，构造方法就应该写以下代码：

```python
def __init__(self, in_channels):
    super(InceptionA, self).__init__()
    # A 
    self.branch_pool = nn.Conv2d(in_channels, 24, kernel_size=1)# 1*1的卷积核

    # B
    self.branch1x1 = nn.Conv2d(in_channels, 16, kernel_size=1)# 1*1的卷积核
    # C
    self.branch5x5_1 = nn.Conv2d(in_channels, 16, kernel_size=1)
    self.branch5x5_2 = nn.Conv2d(16, 24, kernel_size=5, padding=2)
    # D
    self.branch3x3_1 = nn.Conv2d(in_channels, 16, kernel_size=1)
    self.branch3x3_2 = nn.Conv2d(16, 24, kernel_size=3, padding=1)
    self.branch3x3_3 = nn.Conv2d(24, 24, kernel_size=3, padding=1)
```

在初始化后，加入到forward中就可，最后的输出的通道数`24*3 + 16 = 88`

```python
def forward(self, x):
    # A
    branch_pool = F.avg_pool2d(x, kernel_size=3, stride=1, padding=1)# 平均池化
    branch_pool = self.branch_pool(branch_pool)
    # B
    branch1x1 = self.branch1x1
    # C
    branch3x3 = self.branch3x3_1(x)
    branch3x3 = self.branch3x3_2(branch3x3) 
    # D
    branch5x5 = self.branch5x5_1(x)
    branch5x5 = self.branch5x5_2(branch5x5)
    # 组合
    outputs = [branch1x1, branch5x5, branch3x3, branch_pool]
    # cat函数会将各部分组合起来
    # 一共有四个dim：b(batch)、c(channel)、w(weight)、h(height)，这里的1就代表c
    return torch.cat(outputs, dim=1)
```

## Risidual Net

### 梯度消失

在训练过程中可能会遇到**梯度消失**的问题

> 梯度消失：我们嵌套的多层卷积层，如果每个算出的值都是十分接近于0的数，在不断的链式乘积后，就会变得更小，导致梯度消失

解决办法：

【法一】我们可以逐个增加卷积层，去查看哪一层发生了梯度消失

显然这种方法很麻烦

【法二】Risidual Net 残差网络

在基础的NN上，加一个**跳连接**，如图所示

![Risidual Net](http://img.yesmylord.cn//image-20221015215720847.png)

其中，`H(x)= F(x) + x`可以保证`H'(x) = F'(x) + 1`即使`F'(x)`极小，也可以使梯度变化

但是这种Net结构得保证经过卷积后的shape与原shape一样，因此在shape发生变化的位置，我们需要单独处理（处理方式：对x也做处理或是根本不做跳连接）

### 代码实现

```python
class ResidualBlock(nn.Module):
	def __init__(self, channels):
        super (ResidualBlock, self)._init_()
        self.channels = channels
        # 输入和输出通道需要一样
        self.convl = nn. Conv2d(channels, channels, kernel_size=3, padding=1)
		self.conv2 = nn. Conv2d (channels, channels, kernel_size=3, padding=1)
	def forward(self,x):
        y = F.relu(self.conv1(x))
        y = self.conv2(y)
        # 跳连接
		return F.relu(x + y)
```

## 循环神经网络

### RNN出现的原因

有时候会遇到类似于根据上一次的数据，去预测下一次的结果的情况。

比如天气预报，假设我们的数据`温度|湿度|光照->天气`，那么我们根据当天的温度和湿度去预测当天的天气是没有意义的。

> 如何去预测明天的天气呢？

可以将4行记录为一组，3行接在一起作为训练集（`温度1|湿度1|光照1|温度2|湿度2|光照2|温度3|湿度3|光照3`），用第四天的天气作为验证集；这样就可以使用三天的数据去预测后一天的天气了。

> 但是这里存在一个问题，如果单个行记录的参数就很多，那么我们拼凑起来的参数会更多，这样我们在之后的卷积、全连接过程中，可能会参数爆炸
>
> 因此为了解决“参数爆炸”的问题，提出了**循环神经网络RNN**

### RNN Cell

RNN适合去解决数据有序的问题：比如天气预测、比如语言类问题（我|要|去吃饭，语言也有顺序）

RNN的结构如图所示：

![RNN结构](http://img.yesmylord.cn//image-20221025154433654.png)

右图是左图的展开式，可以看到，每一次的计算都需要用到上一次的数据，而参数却还使用原来的同一层，这样可以大大减少计算所需要的参数。

 RNN cell的本质就是一个**线性层**，它的计算式如下：

![RNN cell的计算式](http://img.yesmylord.cn//image-20221025161231100.png)

![RNN cell具体运算](http://img.yesmylord.cn//image-20221025161456208.png)

值得注意的是：xt和ht-1看起来做了两次线性变换，其实我们可以把两个矩阵合起来运算，因此其实就是一次线性运算

比如`w1*h + w2*x = [w1, w2]*[h, x]^T`

### 代码实现

PyTorch中提供的对应框架有两个，可以使用`RNNCell`也可以直接使用`RNN`

- RNNCell

````python
cell = torch.nn.RNNCell(input_size=input_size, hidden_size=hidden_size)
# 只需要提供参数 input_size 和 hidden_size即可
hidden = cell(inputs, hidden)
# RNNcell只有一个输出，就是h1-hn
````

对于`inputs`要注意的是，他就是输入的序列值，他有三个维度`(seq_len, batch_size, input_size)`

对于`hidden`注意，他也有三个维度`(num_layers, batch_size, hidden_size)`

- RNN

```python
cell = torch.nn.RNN(input_size=input_size, hidden_size=hidden_size, num_layers=num_layers)
# 还需要提供 num_layers 即隐藏层的个数，需要几层的RNN
out, hidden = cell(inputs, hidden)
# 使用时给inputs：所有的输入序列、hidden即h0
# 输出会有两个值 out就是h1-hn；hidden就是hn
```

比如一个三层的RNN运算时就是这样的：

![三层的RNN](http://img.yesmylord.cn//image-20221025163804798.png)





