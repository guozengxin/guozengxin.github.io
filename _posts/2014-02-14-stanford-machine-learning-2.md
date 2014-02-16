---
layout: post
title: "机器学习课程总结（二）"
description: "斯坦福大学的机器学习公开课学习笔记。"
category: machine-learning
tags: [机器学习, ]
---
{% include JB/setup %}

### 介绍

本文是机器学习课程笔记的第二篇，从分类问题开始来介绍，包括logistic回归模型。最后介绍了通用线性模型的一些概念。

分类问题和回归问题类似，不同的是在分类问题中，所要预测y的值是一个小量的离散值。在分类问题中，一个典型的问题是二值分类，y的取值只有两个，0 和 1。在这种问题下，样本中y取值为0的样本称为负样本，y取值为1的样本称为正样本。

### Logistic 回归

我们可以先忽略分类的问题离散性，而假设用我们旧的线性回归问题的算法来求解。这里修改回归问题中的假设 $ \\color{black}{h\_\\theta(x)} $ 选择下面的的方程：

$$ \\color{black}{h\_\\theta(x) = g\\left( \\theta\^T x) \\right) = \\frac {1} {1 + e\^{-\\theta\^Tx}}} $$

这里 $\\color{black}{g(z)} $ 被称为**logistic方程**或者**sigmoid方程**：

$$ \\color{black}{g(z) = \\frac {1} {1 + e\^{-z]}}}$$

下面是它的图像：

![logistic方程图像](/assets/images/machine-learing/ml2-1.jpg)

从图像中可以看出，当z远大于0时，$ \\color{black}{g(z)} $的值趋向于1，z远小于0时，$ \\color{black}{g(z)} $的值趋向于0。该函数的这一特征决定了它比较适合0-1分类问题。对于$ \\color{black}{g(z)} $还有一个性质：

$$ \\color{black}{\\begin{align} 
g'(z) \& = \\frac {d}{dz} \\frac {1}{1+e\^{-z}} \\\\
\& = \\frac {1} {\\left(1+e\^{-z}\\right)\^2} \\left( e\^{-z} \\right) \\\\
\& = \\frac {1} {\\left(1+e\^{-z}\\right)} \\cdot \\left( 1 - \\frac {1} {\\left(1+e\^{-z}\\right)} \\right)  \\\\
\& = g(z) \\left(1 - g(z)\\right) \\\\
\\end{align} } $$
 
