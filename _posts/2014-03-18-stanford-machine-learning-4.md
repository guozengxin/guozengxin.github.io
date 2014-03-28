---
layout: post
title: "机器学习课程总结（四）"
description: ""
category: machine-learning
tags: [机器学习]
---
{% include JB/setup %}

### 介绍

机器学习课程笔记的第四篇，讲解支持向量机(Support Vector Machine, SVM)学习算法，SVM是最好的监督学习算法之一。在介绍支持向量机之前，首先介绍**间隔**的概念和把数据用一个巨大的间隔分开；然后，介绍最优间隔分类，这里会引入**拉格朗日对偶**的概念；**核**的引入会使SVM在高维空间的更有效率；最后，会介绍**序列最小优化算法**。

<!-- more -->

### 函数和几何间隔

在介绍边界之前，先了解一个什么是预测的可信度。在logistic回归中，用$ \\color{black}{h\_\\theta(x) = g(\\theta\^T x)} $来预测$ \\color{black}{p(y=1|x;\\theta) }$的值，当$ \\color{black}{\\theta\^T x \\ge 0} $时，预测的结果为1. 但来看训练集中的数据，$ \\color{black}{\\theta\^T x} $越大，$ \\color{black}{h\_\\theta(x)} $的值也越大，此时也就说明该样本标为1具有更高的可信度。同样，当预测时$ \\color{black}{\\theta\^T x \\gg 0} $，说明该输入被预测为1也具有更高的可信度，同理当预测时$ \\color{black}{\\theta\^T x \\ll 0} $，说明该输入被预测为0具有更高的可信度。**可信度**的概念是一个很好的优化优化目标，我们可以使正样本和负样本在模型中都尽量有更高的可信度，那么模型的效果将会更好，我们将用函数间隔来描述这一问题。

对于另一个不同的问题，考虑如下的图形：

![函数间隔](/assets/images/machine-learing/ml4-1.jpg)

在图中，叉号表示正样本，圆圈表示负样本，判定边界($ \\color{black}{\\theta\^T x = 0} $)被称为分隔超平面。另外，在图中可以看到3个点A，B和C。A点离判定边界最远，可以直观上认为在A点标为1非常可信，C点与判定边界非常接近，直观上可以认为C点被判为1的可信度较低（因为几乎都要分到判定边界另一边了）。在这里我们可以用样本点和判定边界的距离来表示样本的可信度，后续将用几何间隔来描述这一问题。


在SVM模型中，为了方便表示，修改之前的表示符号及公式。用$ \\color{black}{y \\in \\{-1, 1\\}} $来表示分类的取值，用参数w，b来表示分类的模型参数：

$$ \\color{black}{h\_{w,b}(x) = g(w\^Tx + b)} $$

其中

$$ \\color{black}{g(z) = \\begin{cases}
1, if z	\\ge 0 \\\\
-1, otherwise
\\end{cases}} $$

#### 函数间隔

给出一个训练样本$ \\color{black}{(x\^{(i)}, y\^{(i)})} $，定义函数间隔是(w,b)的函数：

$$ \\color{black}{\\widehat{\\gamma}\^{(i)} = y\^{(i)} (w\^T x + b)} $$

研究上式，当$ \\color{black}{y\^{(i)} = 1} $时，为了使函数间隔保持一个较大值，需要$ \\color{black}{w\^T x + b} $是一个大的正数；相反的，如果$ \\color{black}{y\^{(i)} = -1} $时，需要$ \\color{black}{w\^T x + b} $是一个大的负数。在预测的时候，如果$ \\color{black}{y\^{(i)}(w\^T x + b) > 0} $，那么这个预测就是一个正确的预测。由此可见，一个大的函数间隔代表一个可信的正确的预测。

对一个训练集$ \\color{black}{S = \\{(x\^{(i)}, y\^{(i)}); i=1,...,m \\} }$，定义参数为 (w, b) 的函数间隔为各样本值中的函数间隔的最小值：

$$ \\color{black}{\\widehat{\\gamma} = \\min\_{i=1,...,m} \\widehat{\\gamma}\^{(i)}} $$

然而，对于线性分类来说，函数间隔并不能很好的描述可信度，原因是模型具有以下特性：模型中，把参数 w 和 b 乘以2，变为 2w 和 2b，并不会改变模型$ \\color{black}{h\_{w,b}(x)} $，但是函数间隔的值会加倍，这意味着模型固定的情况下，函数间隔的值并不是固定的。

接下来，我们来讨论**几何间隔**。

![几何间隔](/assets/images/machine-learing/ml4-1.jpg)

以 (w, b) 为参数判定边界的图形如上所示，w 是一个垂直于判定边界的向量，考虑点A，它代表训练样本$ \\color{black}{(x\^{(i)}, y\^{(i)} = 1)} $，点A到判定边界的距离就叫几何间隔。

怎么去计算几何间隔的值呢？$ \\color{black}{w/||w||} $是和向量 $ \\color{black}{w} $方向相同的单位向量，A点表示为$ \\color{black}{x\^{(i)}} $，可以推出上图中的B点表示为$ \\color{black}{x\^{(i)} - \\gamma\^{(i)} \\cdot w/||w|| } $，由于所有在判定边界上的点都满足$ \\color{black}{w\^T x + b = 0} $，所以:

$$ \\color{black}{w\^T \\left( x\^{(i)} - \\gamma\^{(i)} \\cdot \\frac {w}{||w||} \\right) + b = 0} $$

可以求出几何间隔：

$$ \\color{black}{\\gamma\^{(i)} = \\left( \\frac {w}{||w||} \\right)\^T x\^{(i)} + \\frac {b} {||w||}} $$

上式为正样本的几何间隔的值，事实上，正负样本的几何间隔可以用下式统一表示：

$$ \\color{black}{\\gamma\^{(i)} = y\^{(i)} \\left (\\left( \\frac {w}{||w||} \\right)\^T x\^{(i)} + \\frac {b} {||w||} \\right)} $$

如果$ \\color{black}{||w|| = 1} $，那么几何间隔将和函数间隔相等。对于几何间隔来说，如果将参数 (w, b) 增大为 (2w, 2b)，几何间隔的值是不会变的。

对训练集$ \\color{black}{S = \\{(x\^{(i)}, y\^{(i)}); i=1,...,m \\} }$，定义参数为 (w, b) 的几何间隔为各样本值中的几何间隔的最小值：

$$ \\color{black}{\\gamma = \\min\_{i=1,...,m} \\gamma\^{(i)}} $$

### 最优间隔分类


