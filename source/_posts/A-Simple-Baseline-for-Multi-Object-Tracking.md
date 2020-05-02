---
title: A Simple Baseline for Multi-Object Tracking
date: 2020-04-26 14:57:57
author: Riser
top: true
cover: true
password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
toc: false
mathjax: false
summary: 本文提出了一种在单一网络内部解决多目标跟踪的两个核心组件任务--object detection & re-identication的baseline
categories: CV
tags:
  - MOT
---

### A Simple Baseline for Multi-Object Tracking

#### Abstract

近年来，多目标跟踪的两个核心组件任务--object detection & re-identication

要突出进展，但是使用单一网络解决这个任务的方法研究不多，并且大多由于re-identication分支没有得到充分学习而失败，本文分析失败原因，并且提出来一个baseline，它以30 fps的速度远远超过了公共数据集的最新水平。

**模型代码**：https://github.com/ifzhang/FairMOT.

**Keywords**: One-shot MOT, Simple Baseline, Anchor-free

#### Introduction

多目标跟踪目的：估计视频中多个感兴趣目标的轨迹。

目前先进的做法是将该问题分为两个单独的模型：

- **detection model**

- **association model**

  为每个边界框提取重新标识（Re-ID）特征，并根据在特征上定义的明确的metrics将其链接到一个现有轨道中来。

但是这种方法无法以视频速率推理，因为两部分模型不共享功能。

做一个简单的多任务学习联合这两个模型相较分步处理容易丢失精度，特别是增加了ID switches

本文没有有使用大量技巧来提高跟踪准确性，而是研究了失败的原因，并提出了一个简单而有效的基准。 确定了对跟踪结果至关重要的三个因素。

- **Anchors don't t Re-ID**