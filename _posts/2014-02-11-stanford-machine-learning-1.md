---
layout: post
title: "机器学习课程总结（一）"
description: "斯坦福大学的机器学习公开课学习笔记。"
category: machine-learning
tags: [机器学习, 监督学习, 线性回归, 最小二乘法]
---

### 介绍

机器学习在实际中的应用愈来愈多，也将会成为未来发展的重要方向。由于本人之前对于机器学习的接触都是初步了解，并没有进行系统性的学习。最近打算从斯坦福大学的机器学习公开课开始，从头学习一些机器学习的知识。写一些文章（大部分是课程讲义的翻译）作为自己对于学习知识的巩固，也算是为需要的同学做一个大纲。

本文是第一篇，介绍机器学习的概念、线性回归问题、局部加权线性回归以及相关的一些算法。

<!-- more -->

### 监督学习示例

介绍一个监督学习的例子：一个房价预测问题。考虑如下房价数据：

![房价样本数据](/assets/images/machine-learing/ml1-1.jpg)

我们将通过样本来预测一个房子的价格，使价格作为房间大小的函数，这一过程就成为监督学习。

上图中的房屋大小和房价信息被称为训练集（输入数据），用$\\color{black}{\\left\\{(x\^{(i)}, y\^{(i)}); i=1,2,\\ldots,m \\right\\}}$表示，其中，$\\color{black}{\\left(x\^{(i)}, y\^{(i)}\\right)}$被称为一个训练样本。即将预测的数据被成为目标（输出数据），用y表示。那么监督学习的目标就是，给出一个训练样本，求出一个方程h：$\\color{black}{y=h(x)}$，该方程也被称为一个假设。整个过程如下图所示：

![监督学习过程描述](/assets/images/machine-learing/ml1-2.jpg)

如果需要预测的目标是一个连续值空间，如房价预测，那么这种学习称为**回归问题**，如果y的取值范围是一系列离散值，那么这种学习被称为**分类问题**。

### 线性回归

线性回归的假设为：

$$ \\color{black}{h\_\\theta (x)=\theta\_0 + \theta\_1 x\_1 + \\cdots + \\theta\_n x\_n} $$

其中，$ \\color{black}{\\theta\_i} $ 称为参数。$ \\color{black}{x\_i} $ 代表特征数据，机器学习的目标就是从一些特征预测出所需要的值，例如在房价预测，房屋的大小就是特征。为了表示方便，假设$ \\color{black}{x\_0=1} $，上面的方程可以表示为：

$$ \\color{black}{h\_\\theta(x) = \\sum\_{i=0}\^n {\\theta\_i x\_i}} $$

等号右边的 $\\color{black}{\\theta}$ 和 $\\color{black}{x}$ 均为向量，$\\color{black}{n}$表示输入变量（特征）的数量。

为了实现从训练集求出参数$\\color{black}{\\theta}$的目的，合理的目标就是对于训练集中的所有数据，做到 $ \\color{black}{h(x)} $ 尽可能的等$\\color{black}{y}$。下面的**成本函数**就是为了评价这一目标：

$$ \\color{black}{J(\\theta) = \\frac{1}{2} \\sum\_{i=1}\^m{\\left(h\_\\theta \\left(x\^{(i)}\\right) - y\^{(i)}\\right)\^2} } $$

这一目标函数和**普通最小二乘法**的成本函数相似。

#### LMS算法（最小均方算法）

现在我的目标是求出$\\color{black}{\\theta}$，并且使$ \\color{black}{J(\\theta)} $ 最小化。考虑**梯度下降算法**，给$\\color{black}{\\theta}$ 一个初始值，并重复的修改它的值，使 $ \\color{black}{J(\\theta)} $ 越来越小。

$$ \\color{black}{\\theta\_j := \\theta\_j - \\alpha \\frac {\\partial}{\\partial \\theta\_j} J(\\theta) } $$

这里的 $ \\color{black}{\\alpha} $ 称为学习速度，$ \\color{black}{\\alpha} $ 越大，算法收敛的越快，反之越慢。上式中对 $ \\color{black}{J(\\theta)} $ 求偏导之后（求解过程省略）：

$$ \\color{black}{\\frac {\\partial}{\\partial \\theta\_j} J(\\theta) = \\left( h\_\\theta (x) - y\\right) x\_j} $$

那么更新方程变为：

$$ \\color{black}{\\theta\_j := \\theta\_j + \\alpha \\left(y\^{(i)} - h\_\\theta \\left( x\^{(i)} \\right) \\right) x\_j\^{(i)} } $$

这个更新方程可以很直观的看出它的意义，对于**误差项** $\\color{black}{\\left( y\^{(i)} - h\_\\theta \\left( x\^{(i)} \\right) \\right)} $来说，如果一个样本根据假设得到的值和实际值相差很小，那么参数在下一步更新也很小，反之如果根据假设得到的值和实际值相差较大，那么下一步参数更新的幅度也较大。

衍生出多个样本的梯度下降算法：**批量梯度下降算法**

>Repeat until covergence {

>    $ \\color{black}{\\theta\_j := \\theta\_j + \\alpha \\sum\_{i=1}\^{m} \\left( y\^{(i)} - h\_\\theta\\left( x\^{(i)} \\right) \\right) x\_j\^{(i)}} $

>}

这个算法每迭代一次，所有样本都会生效。虽然梯度下降算法有可能会导致局部最优而不是全局最优，但是在线性回归问题中，只有一个全局最优解，而没有局部最优的可能。算法检验收敛的方法通常是检验两次迭代中参数的改变程度或者成本函数的改变程度，当两次迭代中改变程度小到一定阈值时，认为算法收敛。

当训练集很大时，对于批量梯度下降算法，每一步更新都要计算大量的样本。此时应该使用**随机梯度下降**算法：

>Loop {

>    for i=1 to m {

>   $ \\color{black}{\\theta\_j := \\theta\_j + \\alpha \\left(y\^{(i)} - h\_\\theta \\left( x\^{(i)} \\right) \\right) x\_j\^{(i)} } $        ( for every j)

>    }

>}

#### 其它求解过程

在课程中，Andrew Ng 用矩阵运算回顾了最小二乘法，并直接求出参数的解：

$$ \\color{black}{\\theta = \\left( X\^T X \\right)\^{-1} X\^T \overrightarrow {y} } $$

教授用概率的方法对线性回归问题进行了解释，采用**最大似然估计**的方法可以得到与最小二乘法完全相同的成本函数。

### 局部加权线性回归

线性回归将真实数据拟合成了一个线性的模型，当真实数据并不符合线性规律时，这种方法的预测会带来偏差。看看这个图：

![非线性样本数据](/assets/images/machine-learing/ml1-1.jpg)

直线的拟合并没有很好符合真实数据，第二幅图是一个二次函数的图形，它比直线较好的符合真实数据，但是还不是完全符合。最右边的图形是一个5次的函数，它很好的描述了真实的数据。这里有两个概念需要说明一下，**欠拟合**：模型没有很好的捕捉到训练数据的结构；**过拟合**：模型假设的过于复杂，对训练数据的捕捉很好，但是却没有真正描述真实世界中的数据结构。

对于非线性的模型，一个合理的方法是使用**局部加权线性回归**算法，它假设所预测的点附近很短一段距离是符合线性的。在之前的线性回归模型中，我们的做法是：寻找合适的 $ \\color{black}{\\theta} $ 使 $ \\color{black}{\\sum\_i \\left( y\^{(i)} - \\theta\^T x\^{(i)} \\right)\^2} $ 最小。现在在局部加权线性回归模型中，需要寻找合适的 $ \\color{black}{\\theta} $ 使 $ \\color{black}{\\sum\_i w\^{(i)} \\left( y\^{(i)} - \\theta\^T x\^{(i)} \\right)\^2} $ 最小。这里，$ \\color{black}{w\^{(i)}} $ 是一个非负的权值，直观来看，$ \\color{black}{w\^{(i)}} $ 越大，对模型的影响越大，$ \\color{black}{w\^{(i)}} $ 越小，对模型的影响越小。通常用下面的函数：

$$ \\color{black}{w\^{(i)} = exp \\left( -\\frac {\\left( x\^{(i)} - x \\right)\^2} {2 \\tau\^2} \\right)} $$

从这个函数中可以看出，如果 $ \\color{black}{\\left| x\^{(i)} - x \\right|} $ 越大，$ \\color{black}{w\^{(i)}} $ 越接近 1，反之如果$ \\color{black}{\\left| x\^{(i)} - x \\right|} $ 越小，$ \\color{black}{w\^{(i)}} $ 越接近 0。$ \\color{black}{\\tau} $ 称为波长参数，它反应了权值随距离下降的速率。

局部加权线性回归是一个**非参数学习算法**，相比而言，线性回归是一个**参数学习算法**，因为它需要求解一个固定且有限的参数 $ \\color{black}{\\theta\_i} $ ，而且求模型训练完成后就不再需要训练数据了。但局部加权线性回归在预测时始终需要训练数据来参与计算。

### 小结

本节介绍了机器学习的概念，线性回归模型、最小均方算法、梯度下降算法、局部加权线性回归模型等。这些是一个基本算法，对于理解机器学习的概念和目标是很好示例。省略了矩阵运算和概率解释的推导过程（公式太多且复杂...）.


### 参考

1. 斯坦福大学公开课--机器学习课程: <http://v.163.com/special/opencourse/machinelearning.html>
2. 课程课件下载地址: <http://cimg3.163.com/edu/open/ocw/jiqixuexikecheng.zip>
3. 最小二乘法: <http://zh.wikipedia.org/wiki/%E6%9C%80%E5%B0%8F%E4%BA%8C%E4%B9%98%E6%B3%95>
4. 最大似然估计: <http://zh.wikipedia.org/wiki/%E6%9C%80%E5%A4%A7%E4%BC%BC%E7%84%B6%E4%BC%B0%E8%AE%A1>
