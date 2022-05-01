---
layout: post
title: 【论文阅读】Sharpness-Aware Minimization
categories: SAM
description: 神经网络的优化
keywords: Deeplearning, Optimization
---
SAM的论文地址：[Sharpness-Aware Minimization for Efficiently Improving Generalization](https://arxiv.org/pdf/2010.01412.pdf)
普通的防止过拟合我脑子里浮现的方法又比如增加正则化，数据增强，dropout等等.
SAM是全新的防止过拟合方法, 我想先从以下三个方面浅谈一下我对这篇paper的认识。
- 什么是 SAM 以及动机
- 工作原理
- CIFAR10和Fashion-MINIST上的实验结果

什么是 SAM 以及动机：
sam本质上是一个优化器去同时 最小化loss funtion 和 sharpness 从而去或得一个相对平坦的损失图. 在过去的两年里比较受欢迎也是因为比较容易在代码阶段实现 而且不会大幅度增加computation cost. 而且也在当时许多 公开的数据集上比如 imagenet和cifar上SOTA.
左边是SGD的在resnet上训练的 loss lanscape， 右边是加 SAM+SGD的. 可以看出得到一个更加平滑的loss lanscape. 作者认为右边这种拥有更加平滑的minimum的模型会有更强的泛化性，因为可以防止陷入local minimum. hello hello
![](https://github.com/calvin886/calvin886.github.io/blob/master/images/blog/SAM_loss_landscape.jpg?raw=true)
