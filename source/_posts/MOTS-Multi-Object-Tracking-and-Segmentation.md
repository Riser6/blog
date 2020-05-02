---
title: 'MOTS: Multi-Object Tracking and Segmentation'
date: 2020-04-13 10:32:35
author: Riser
top: true
cover: true
password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
toc: false
mathjax: ture
summary: 本文介绍一篇基于分割的多目标跟踪任务的论文，并介绍论文所给出的数据集，评价指标和基线方法
categories: CV
tags:
  - MOTS
  - video segmentation
---

### MOTS: Multi-Object Tracking and Segmentation

#### Introduction

##### 背景

多目标跟踪很难，bounding_box跟踪性能达到饱和， 进一步的改进需要移动到像素级别。因此，我们建议将检测、分割和跟踪三个任务视为需要一起考虑的相互关联的问题 

##### 必要性

1.基于bounding_box的跟踪过于粗糙，特别是在目标重叠的情况下，特定目标的边界框中来自其他目标的信息甚至要超过自身的，这种情况下使用分割更加自然，可以提供更多信息给后续过程。

2.实际可能有多个bounding_box大概都能装下特定目标，容易产生歧义，具体与ground truth进行比较时，还需要额外的匹配程序，而分割产生的掩模可以直接与ground truth。

因此作者提出来Multi-Object Tracking and Segmentation (MOTS)，并使用TrackR-CNN作为baseline处理MOTS任务，TrackR-CNN扩展了带3D卷积的MaskR-CNN来合并时间信息，并通过一个关联头来链接对象身份随着时间推移。

##### 主要贡献

1.为处理MOTS任务制作了KITTI和MOTChallenge两个数据集

2.提出来针对MOTS任务的评价方法sMOTSA

3.使用TrackR-CNN作为一种解决检测跟踪和分割的baseline方法，并与现有的工作进行比较

4.证明了新数据集对像素级多目标跟踪器端到端训练的有效性，证明分割和跟踪联合训练的可能性

#### Datasets

 为视频中每个对象的每一帧注释像素掩码。

##### 半自动注释的程序 

提出一种通过分割掩码来扩展bounding box级别标注的半自动方法。使用卷积网络DeepLabv3+从bounding box中自动生成分割Mask，然后使用手动多边形标注进行校正。对于每条轨迹，都使用手动标注作为附加训练数据来微调初始网络。一直迭代生成和校正Mask过程,直到达到标注的像素级精度。

##### KITTI MOTS

进行上述步骤，并将数据集划分为训练集和验证集，并且进行类平衡，注释过程手动工作比较大，表明基于单张图片的分割技术有限，这也是作者想要在分割模型中融合时间因素的动机。

##### MOTSChallenge

 MOTSChallenge主要关注拥挤场景中的行人，由于有很多遮挡的情况，所以非常具有挑战性，像素化的描述尤其有用 

#### Evaluation Measures

 要求物体的ground truth掩模和MOTS方法产生的掩模均不重叠 ，即 每个像素只分配给一个object 。

详见原论文

#### Method

TrackR-CNN是在MaskR-CNN的结构基础上做的改进，下图中黄色的部分是前者不同于后者的部分。

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200402144017.png)

TrackR-CNN 通过一个关联头（association head）和两个3D卷积层来扩展Mask R-CNN ，使得可以产生基于时间序列的关联检测，然后将TrackR-CNN的基于掩模探测结果和关联检测结果输入到追踪算法，由算法来根据时序上的关联度将不同帧的探测结果链接起来，得到满足要求的视频多目标追踪输出结果。

##### Integrating temporal context

 将3D卷积（其中第三个维度是时间）整合到以ResNet-101 为骨干网的Mask R-CNN中 ，网络接收多帧图片输入；  作为替代方案，还考虑卷积LSTM 层（卷积LSTM通过使用卷积而不是矩阵乘积计算其激活来保留输入的空间结构）。 本质上的是为了得到整合了时序信息的增强特征喂给RPN网络。

##### Association Head

关联头实质上是一个编码器，由一个全连接层获取区域建议作为输入，编码产生128维的关联向量，以该关联向量代表视频中目标的身份，以属于同一实例的向量彼此接近，属于不同实例的向量彼此远离的原则训练该编码器。  以欧几里德距离定义两个关联向量v和w之间的距离$d(v,w)$ ，即 

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200402151258.png)

使用batch hard triplet loss 来训练关联头使其适用于视频序列。让D表示视频的检测集。 每个检测$d$ ∈$D$由掩模$mask_d$和关联矢量$a_d$组成，其来自时间帧$t_d$，由与之重叠的ground-truth目标给定对应的ground-truth跟踪id$id_d$ 。 对于T时间步长的视频序列，具有边际α的batch-hard formulation中的关联损失由下式给出 

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200402151305.png)

##### Mask Propagation

原文对这一部分的描述略显抽象，我们首先需要弄清什么是光流，实质上它表示因为目标与摄像头的一个相对运动导致同一目标在视频中不同帧的图像中出现的具体坐标区域有差异，而这一部分的工作就是利用光流信息将目标在不同帧图像中的坐标位置差异抹除，然后计算不同帧产生的掩码彼此之间的IOU作为上面关联信息的另一个替代线索，具体做法是

对于在时间t-1处的检测d∈D具有掩模$mask_d$并且在时间t的检测e∈D处具有掩模$mask_e$我们定义掩模的传播得分为

![](https://gitee.com//Riser_Wu/images/raw/master/img/20200402152412.png)

其中W(m)表示原处于t0处的掩模m经过光流信息处理后的对应到t1处新产生的掩模。

##### Tracking

我们需要清楚，TrackR-CNN网络仅仅提供了多目标在不同帧间的掩模探测结果和时序上的关联度信息，接下来需要我们的跟踪算法基于给定的关联度信息链接不同帧的探测结果，然后给出跟踪要求的输出结果。

实质上还是通过设立关联度阈值来整合单帧探测结果到追踪结果（调参。。），比较重要的是，论文提到了为了使最终的结果不出现目标重叠现象，我们对结果中像素重叠的部分采取探测结果置信度高优先的原则分配重叠的pixels部分

#### Experiments&Conclusion

略