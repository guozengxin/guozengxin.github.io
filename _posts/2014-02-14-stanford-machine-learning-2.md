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

这个模型中，如何求解 $ \\color{black}{\\theta} $ 呢？这里采用最大似然估计来求解，为模型做一系列概率假设。

#### 似然函数求解（一)

假设：

$$ \\color{black}{\\begin{alignat} {2}
P(y=1|x;\\theta) \& = h\_\\theta(x) \\\\
P(y=0|x;\\theta) \& = 1 - h\_\\theta(x) \\\\
\\end{alignat}} $$

两式合起来：

$$ \\color{black}{ p(y|x;\\theta) = \\left(h\_\\theta(x)\\right)\^y \\left( 1 - h\_\\theta(x) \\right)\^{1-y}} $$

假设 m 个训练样本是相互独立的，则可以有如下的似然函数：

$$ \\color{black}{\\begin{align}
L(\\theta) \& = p\\left( \\vec{y}|X;\\theta \\right) \\\\
\& = \\prod\_{i=1}\^m p\\left( y\^{(i)}|x\^{(i)}; \\theta \\right) \\\\
\& = \\prod\_{i=1}\^m \\left( h\_\\theta \\left(x\^{(i)}\\right) \\right)\^{y\^{(i)}} \\left( 1 - h\_\\theta \\left(x\^{(i)}\\right) \\right)\^{1-y\^{(i)}} \\\\
\\end{align} } $$

现在令似然函数最大化：

$$ \\color{black}{\\begin{align}
l(\\theta) \& = \\log L(\\theta) \\\\
\& = \\sum\_{i=1}\^m y\^{(i)} \\log h\\left(x\^{(i)} \\right) + \\left(1-y\^{(i)}\\right) \\log \\left( 1 - h\\left( x\^{(i)} \\right) \\right) \\\\
\\end{align} } $$

现在用梯度下降算法，$ \\color{black}{\\theta := \\theta + \\alpha \\nabla\_\\theta l(\\theta) } $，（因为目标是最大化目标函数，所以这里是加号）。对 $ \\color{black}{l(\\theta)} $求偏导数(代入$ \\color{black}{g'(z) = g(z)\\left(1 - g(z) \\right)} $，过程省略)：

$$ \\color{black}{ \\frac {\\partial} {\\partial \\theta\_j} l(\\theta) = \\left(y - h\_\\theta(x) \\right) x\_j } $$

这样，我们就可以通过一个随机梯度下降算法求得$ \\color{black}{\\theta} $的值：

$$ \\color{black}{\\theta\_j := \\theta\_j + \\alpha \\left( y\^{(i)} - h\_\\theta \\left( x\^{(i)} \\right) \\right) x\_j\^{(i)}} $$

这个更新函数和**最小均方算法**的更新函数非常相似，但是完全不同，因为这个模型是一个非线性的方程。

#### 似然函数求解（二）牛顿方法

牛顿方法和梯度下降类似，都是逐渐对空间进行搜索的方法。例如有一个函数 $\\color{black}{f: \\mathbb{R} \\rightarrow \\mathbb{R} }$, 我们的目标是求出 $\\color{black}{\\theta}$，使 $ \\color{black}{f(\\theta) = 0} $，下面是更新函数：

$$ \\color{black}{\\theta := \\theta - \\frac {f(\\theta)} {f'(\\theta)} } $$

牛顿方法的更新步骤如下图所示：

![牛顿方法的更新步骤](/assets/images/machine-learing/ml2-2.jpg)

随机选一个点找出这个点的切线，延长切线与x轴相交，交点的x值作为下一次迭代时x的值。

那么如何用牛顿方法来使$ \\color{black}{l(\\theta)} $最大化呢？其实使 $ \\color{black}{l(\\theta)} $ 最大化时 $ \\color{black}{l(\\theta)} $ 的导数为0. 那么就和上述问题一致了，因此更新函数为：

$$ \\color{black}{\\theta := \\theta - \\frac {l'(\\theta)}{l''(\\theta)} } $$

在logistic回归问题中，$ \\color{black}{\\theta} $ 实际上是一个向量，所以需要对牛顿方法进行扩展：

$$ \\color{black}{\\theta := \\theta - H\^{-1} \\nabla\_\\theta l(\\theta) } $$

$ \\color{black}{\\nabla\_\\theta l(\\theta)} $ 是 $ \\color{black}{l(\\theta)} $ 的偏导数，H 被称为 Hessian 矩阵，是$ \\color{black}{l(\\theta)} $的二次偏导数矩阵。其中 $ \\color{black}{H\_{ij}} $ 计算如下：

$$ \\color{black}{H\_{ij} = \\frac {\\partial\^2 l(\\theta)} {\\partial \\theta\_i \\theta\_j}} $$

牛顿方法的特点就是收敛比较快，需求较少的迭代次数，但是由于需要计算 H 的逆，所以在每一次迭代中耗时非常高。然而只要 n 不是太大，牛顿方法还是一个非常快的方法。如果用牛顿方法来求最小值时，更新函数不变。当二阶导数小于0时，求的最大值，当二阶导数大于0时，求的最小值。

### 通用线性模型

在之前介绍的回归例子中，模型服从正态分布：$ \\color{black}{y|x;\\theta ~ N\\left( \\mu, \\sigma\^2 \\right)} $，在分类示例中，模型服从伯努力分布：$ \\color{black}{y|x;\\theta ~ Bernoulli(\\phi)} $。在通用线性模型中，将会展示这些模型共有的特点。

#### 指数族

指数族分布的概念，对于分布函数能够用如下方式表现的分布，称为指数族分布：

$$ \\color{black}{p(y;\\eta) = b(y)exp\\left(\\eta\^T T(y) - a(\\eta) \\right)} $$

这里，$ \\color{black}{\\eta} $是自然参数(natural parameter)，也叫做标准参数(canonical parameter)，T(y)被称为充分统计量，$ \\color{black}{a(\\eta)} $称为对数配分函数(log partition function)，$ \\color{black}{e\^{-a(\\eta)}} $ 的值实质上扮演了一个归一化的角色，保证了分布的和为1.

在 T, a 和 b 固定之后，我们就得到了一个以 $ \\color{black}{\\eta} $为参数的指数分布。

接下来将展示伯努力分布和高斯分布都可以用指数族分布的形式来表示。
