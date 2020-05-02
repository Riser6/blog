---
title: Learning Lightweight Lane Detection CNNs by Self Attention Distillation
date: 2020-04-13 12:14:47
author: Riser
top: true
cover: true
password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
toc: false
mathjax: false
summary: 本文介绍一种自注意蒸馏的方式用于提升轻量级网络性能，原论文的应用场景是无人驾驶领域的车道线检测，本文给出对这一工作的具体描述和解决方案，并与平行的几个方案进行了对比。
categories: CV
tags:
  - SAD
  - automatic drive 
  - Lane Detection
---

### Learning Lightweight Lane Detection CNNs by Self Attention Distillation

#### Abstract

在监督稀疏的情况下训练车道线检测的精确的深度学习模型很难，本文提出了一种叫做自注意蒸馏（Self Attention Distillation）的知识蒸馏方法（knowledge distillation），它使得模型能不借助另外的监督标签下通过自我学习提升。另外发现模型的内部attention maps含有丰富的上下文信息，可以利用这些信息作为监督进行自上而下的逐层attention提取。且容易合并到任何前馈学习的CNN中，不增加inference time,本文的轻量级模型ENet-SAD与当前最先进的模型相比，参数量少20倍，速度快10倍。

#### Introduction

背景

车道线检测的固有难题

1. 昏暗的光照条件
2. 车辆堵塞的情况
3. 不相关的道路标记
4. 车道线的细长特性

其中由于目前车道线检测主要是基于分割任务，但是由于车道线的细长特性，它在一张图片中的有效占比很少（相较于大部分无效的背景），给分割任务带来很大的挑战，一种做法是增加车道线注释的宽度，但它会影响探测性能。

现有解决方案

1. 多任务学习(MTL）

2. 信息传递（MP）

   这两个方案都有代价，MTL需要额外的标注，MP需要增加inference time

本文的方案(SAD)

优点：不需要额外的标签监督，不增加inference time

动机：当模型训练到一定水平时，特征图会对车道线的位置和轮廓产生较丰富的信息，见下图中的上半部分。同时也要注意基本episode40k~60k没有太多改进，基本达到饱和。

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200404102219.png)

做法：我们使用低层block的attention maps去模仿深层block的attention maps

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200404103709.png)

效果：

- 低层的attention maps变得更加精细（保留了更多的原图的一些边缘轮廓信息）
- 低层更好的特征表达造福于深层

我们可以利用SAD训练具有优秀注意力的小网络，并且达到与深度网络匹配的性能。

贡献

- 我们提出了一种新的注意力蒸馏方法，即SAD，以增强基于CNN的车道检测模型的表示学习。SAD只在训练阶段使用，在部署过程中不会带来计算成本，我们的工作是首次尝试使用网络的注意力图作为蒸馏目标。
- 我们仔细而系统地研究了SAD的内在机制，考虑在不同层次的模拟路径中进行选择，以及将SAD引入培训过程以提高收益的时间点。
- 我们验证了SAD对提高小车道检测网络性能的有用性。

#### Related Work

##### 知识和注意力蒸馏（Knowledge and attention distillation）

是一种轻化网络的做法

通常是一个学生网络向另外一个老师网络学习，通过模仿attention map来训练学生网络，使得小网络具备大网络的性能，相同维度网络间的学习和异构网络间的学习都有对应的研究，我们的方法是在单个网络间进行自上而下的逐层自我学习。需要额外注意的是，这个过程只在训练过程中进行，那么自然不会增加它的inference time（实质上就是将部署到芯片的网络参数变得更加科学，inference步骤都不变）。

#### Methodology

##### Self Attention Distillation

attention maps分为activation-based attention maps和
gradient-based attention maps（二者区别为是否使用激活函数），本文指出基于激活值的attention maps收获更好的表现（另外一种几乎不起作用）

###### Activation-based attention distillation

实质上是将多通道的3D激活输出压缩转化为2D表达，即（m表示层数）

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200408144546.png)

映射结果中每个元素的绝对值表示此元素对最终输出的重要性 

文章就此映射尝试了三种方案，如下：

![](https://gitee.com//Riser_Wu/images/raw/master/img/4688102-ed52d5b5a80b3cf1.png)

实验发现第二种方案更好(第二张与第三种比较明显高激活区域权值更大，而个人感觉第二张与第三张的比较主要是利用平方对小的背景干扰的一个弱化)，并设置p=2

![](https://gitee.com//Riser_Wu/images/raw/master/img/4688102-d811a46d1c937435.png)

###### 网络结构

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200408154205.png)

网络结构很简单，见上。

论文中提到了一点改进： 在不增加参数数量的情况下，增加扩容卷积来代替P1中的原来的卷积层，增加网络的接受域。 

###### Adding： SAD to training

- ATGEN功能：

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200408155210.png)

β是双线性插值（统一尺度），φ是softmax运算（统一量纲）

- 蒸馏损失公式

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200408155624.png)

$L_d$是$L_2$损失，这里损失是多层的直接求和，可以但没有设立权重参数（估计没太大意义）

- 整体损失公式

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200408160148.png)

 segmentation loss由交叉熵函数和IoU（论文中写的有点奇怪）组成；existence loss为二分类交叉熵函数。 

- 不太重要的

  模仿的路径多种（类似Densenet）有多种，并且做了有无SVD效果的概率图可视化工作：

![](https://gitee.com//Riser_Wu/images/raw/master/img/4688102-2bcfdff91932ed15.png)

##### Lane Prediction

 对输出的概率图先进行9x9卷积的光滑处理，显示取阈值大于0.5的像素，然后每隔20行取最大响应值，最后三次样条插值连接这些最大响应值。 （不是本文主要工作，写的很简略，具体参考汤晓鸥老师组的X.Pan,J.Shi,P.Luo,X.Wang,andX.Tang.Spatialasdeep: Spatial CNN for traffic scene understanding. In Association for the Advancement of Artificial Intelligence, 2018.）

#### Experiments

数据集，评价指标，实现细节及结果自见论文

##### Ablation Study

SVD路径及方向

![](https://gitee.com//Riser_Wu/images/raw/master/img/4688102-0212663c622ecbd9.png)

- SVD在中高层工作更好
- 在低层加入SVD会降低性能，理由：因为网络低层本来是用来捕获细节信息，模仿高层结果会丢失这些信息
- 相邻层连接的效果更好，理由：相邻层本身在特征上也要更加相似
- 自高向低的模仿路径行不通，理由：低层有很多细节信息(相对于本任务最终输出是噪声信息)

###### SVD VS 深度监督

![](https://gitee.com//Riser_Wu/images/raw/master/img/4688102-d6da35da1a707c88.png)

深度监督：直接使用标签作为每层的监督

深度监督可以带来性能的提升，但是比不上SAD

###### When to add SAD

![](https://gitee.com//Riser_Wu/images/raw/master/img/4688102-d699a23400668711.png)

加入SVD的时间点对最终性能影响不大，但是对网络收敛速度有影响，建议在原网络训练后期加入，感觉可能加入SVD网络的学习速率会降低，所以在网络训练差不多完成时加入较好。

