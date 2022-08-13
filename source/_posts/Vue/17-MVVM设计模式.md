---
title: 17.MVVM设计模式
date: 2019-07-24 21:36:50
tags: 
- Vue
categories: 
- 前端
- Vue
---
<center>
引言：

MVVM设计模式到底是什么呢？

它和传统的MVP设计模式相比优点是什么呢?

</center>

<!--more-->
----------
# 传统的MVP设计模式

M: Model数据层

V: View视图层

P: Prestenter呈现层，控制层

**M  <--> P <--> V**

视图层V发生事件，交给控制层P,核心层是P

代表语言: `jquery`

他们之间依次交流，交换信息，简化了DOM操作，但还是没有解决"DOM无关于业务逻辑，不应该出现在编辑的代码中的问题"

逐步淘汰

# MVVM设计模式

M: Model数据层

V: View视图层

VM: ViewModel(桥梁:不需要关注这层怎么实现)

![image](https://images2018.cnblogs.com/blog/1287630/201711/1287630-20171127152201940-1604530014.png)


