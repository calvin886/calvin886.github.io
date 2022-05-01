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
左边是SGD的在resnet上训练的 loss lanscape， 右边是加 SAM+SGD的. 可以看出得到一个更加平滑的loss lanscape. 作者认为右边这种拥有更加平滑的minimum的模型会有更强的泛化性，因为可以防止陷入local minimum.
![](https://github.com/calvin886/calvin886.github.io/blob/master/images/blog/SAM_loss_landscape.jpg?raw=true)

工作原理:
另外一个有点就是，代码比较易懂，容易复现，现在先简单介绍一下代码，我们可以看到第二行是计算我们的常规对损失函数求梯度，然后 接下来的一行计算梯度的norml, 然后更新$w_t$沿着$e_hat$的方向找到$w_adv$，然后算$w_adv$ 的梯度 然后在更新$w_t$到$w_t+1$. 这就是一个代码角度比较简洁的梯度更新的过程
![](https://github.com/calvin886/calvin886.github.io/blob/master/images/blog/pcode_for_sam.jpg?raw=true)

数学上:左边$L_d$是我么的expected loss，true loss，右边$L_s$是trainingloss也是emprical loss. 右边 的公式我们可以理解为，现在给我们一个parameter w，我们想在一个半径是rho的球形的constarin里面寻找一个 worst loss，最大的loss. 右边第二项是在papr的附录里摘抄出来的，很长 ，但是好在唯一的变量是一个w norm。而且是一个单调递增的函数 with $w_norm$/tho. 然后作者吧h代替成了lambda w norm 的平方方便证明,让整体成为了一个正则项.
然后我们简单的减加一下Ls，让左边的一合并就得出了sharpness的形式. 他表示了在w附近的最 大的loss change，在w附近的loss lanscap越陡，这一项就越大. 所以在整个这个式子里就是同时 minimize 第一项sharpness，第二项regualr loss，和 正则项。
![](https://github.com/calvin886/calvin886.github.io/blob/master/images/blog/page4.jpg?raw=true)
接下来利用泰勒一阶导，和对偶范数展开.
![](https://github.com/calvin886/calvin886.github.io/blob/master/images/blog/page5.jpg?raw=true)
![](https://github.com/calvin886/calvin886.github.io/blob/master/images/blog/page6.jpg?raw=true)

红色的是一阶导数,用Chain rule求的，先算括号里对w的导数，求出右边这个d(w+e(w))/dw -> Identitiy function, 然后外面再乘上Loss的对w的导数。
蓝色的是二阶导数,因为e_hat(w) 包含了对Ls的导数，所以对导数求导，这里会出现一个二阶导。 但是作者说带上蓝色的二街导会让performance 变差. 之后去掉了
![](https://github.com/calvin886/calvin886.github.io/blob/master/images/blog/page7.jpg?raw=true)

梯度优化方面我之前提了一下代码，现在想从这个参数更新图上在介绍一下, 从黄色线$w_t$到$w_t+1$ 是常规的梯度更新，然这$w_t$的方向更新一个step。 加上sam，现在我们要找到一个最大的loss值在 $w_t$的邻域里，方向是$e_hat(w)$的方向。然后我们算$w_adv$的梯度，最后更新 wt在$w_adv$的方向上，最终得$wt+1_SAM$.
![](https://github.com/calvin886/calvin886.github.io/blob/master/images/blog/page8.jpg?raw=true)

在开源代码上的复现测试:
![](https://github.com/calvin886/calvin886.github.io/blob/master/images/blog/page9.jpg?raw=true)
