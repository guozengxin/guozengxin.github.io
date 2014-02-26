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

我们可以先忽略分类问题的离散性，而假设用我们旧的线性回归问题的算法来求解。这里修改回归问题中的假设 $ \\color{black}{h\_\\theta(x)} $， 选择下面的的方程：

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

#### 似然函数求解（一）

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

现在求解对数似然并使其最大化：

$$ \\color{black}{\\begin{align}
l(\\theta) \& = \\log L(\\theta) \\\\
\& = \\sum\_{i=1}\^m y\^{(i)} \\log h\\left(x\^{(i)} \\right) + \\left(1-y\^{(i)}\\right) \\log \\left( 1 - h\\left( x\^{(i)} \\right) \\right) \\\\
\\end{align} } $$

用梯度下降算法，$ \\color{black}{\\theta := \\theta + \\alpha \\nabla\_\\theta l(\\theta) } $，（因为目标是最大化目标函数，所以这里是加号）。对 $ \\color{black}{l(\\theta)} $求偏导数(代入$ \\color{black}{g'(z) = g(z)\\left(1 - g(z) \\right)} $，过程省略)：

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

牛顿方法的特点就是收敛比较快，需求较少的迭代次数，但是由于需要计算 H 的逆，所以在每一次迭代中耗时非常高。然而只要 n 不是太大，牛顿方法还是一个非常快的方法。如果用牛顿方法来求最小值时，更新函数不变。当二阶导数小于0时，求出的是最大值，当二阶导数大于0时，求出的是最小值。

### 通用线性模型

在之前介绍的回归例子中，模型服从正态分布：$ \\color{black}{y|x;\\theta \\sim N\\left( \\mu, \\sigma\^2 \\right)} $，在分类示例中，模型服从伯努力分布：$ \\color{black}{y|x;\\theta \\sim Bernoulli(\\phi)} $。在通用线性模型中，将会展示这些模型共有的特点。

#### 指数族

指数族分布的概念，对于分布函数能够用如下方式表现的分布，称为指数族分布：

$$ \\color{black}{p(y;\\eta) = b(y)\\exp\\left(\\eta\^T T(y) - a(\\eta) \\right)} $$

这里，$ \\color{black}{\\eta} $是自然参数(natural parameter)，也叫做标准参数(canonical parameter)，T(y)被称为充分统计量，$ \\color{black}{a(\\eta)} $称为对数分配函数(log partition function)，$ \\color{black}{e\^{-a(\\eta)}} $ 的值实质上扮演了一个归一化的角色，保证了分布的和为1.

在 T, a 和 b 固定之后，我们就得到了一个以 $ \\color{black}{\\eta} $为参数的指数分布。

接下来将展示伯努力分布和高斯分布都可以用指数族分布的形式来表示。

#### 伯努力分布

伯努力分布可以表示成如下形式：

$$ \\color{black}{p(y;\\phi) = \\phi\^y(1-\\phi)\^{1-y}} $$

变换成类似指数族分布的形式：

$$ \\color{black}{exp\\left( \\left( log \\left( \\frac{\\phi}{1-\\phi)} \\right) \\right)y + log(1-\\phi) \\right)} $$

在上式中，自然参数$ \\color{black}{\\eta = log\\left(\\phi/(1-\\phi)\\right)} $，求解出$ \\color{black}{\\phi = 1/(1+e\^\\eta)} $，这个函数与sigmoid函数相似！通过代换，得到指数族分布的其它参数：

$$ \\color{black}{\\begin{align}
T(y) \&= y \\\\
a(\\eta) \&= - log (1-\\phi) \\\\
	\&= log(1+e^\\eta) \\\\
b(y) &= 1
\\end{align} } $$

由高斯分布可以推导出线性模型，但是其方差与模型的解无关，所以将高斯分布的方差$ \\color{black}{\\eta\^2} $设为1。因此高斯分布可以作如下转换：

$$ \\color{black}{\\begin{align}
p(y;\\mu) \&= \\frac {1} {\\sqrt{2\\pi}} exp \\left( - \\frac {1}{2} (y-\\mu)\^2 \\right) \\\\
\&= \\frac {1} {\\sqrt{2\\pi}} exp \\left( - \\frac{1}{2} y\^2 \\right) exp \\left(\\mu y - \\frac{1}{2}\\mu\^2 \\right)
\\end{align} } $$

对应的指数族分布各参数如下：

$$ \\color{black}{\\begin{align}
\\eta \&= \\mu \\\\ 
T(y) \&= y \\\\
a(\\eta) \&= \\mu\^2 / 2 \\\\
\&= \\eta\^2 / 2 \\\\
b(y) \&= \\left(1/\\sqrt{2\\pi} \\right) exp \\left(- y\^2 / 2 \\right) \\\\
\\end{align} } $$

还有很多常见的分布都属于指数族分布，如

1. 多项式分布
2. 波松分布
3. $ \\color{black}{\\gamma} $分布
4. $ \\color{black}{\\beta} $分布
5. 指数分布
6. 狄里克雷分布

### 广义线模型

假如我们要估计进入一个商店的人数的分布，特征可以有天气、时间、推广...等等，比较合适的一个分布就是波松分布。由于波松分布也属于指数分布，对波松分布的求解也可以归结为对指数族分布的求解。这里需要构造一个**广义线性模型**(GLM, Generalized Linear Model)。

更广泛的来讲，对于回归问题或者分类问题来说，我们需要预测一个随机变量y作为x的函数。在广义线性模型中，我们为给定x的条件下y的分布作以下的假设：

1. $ \\color{black}{y|x;\\theta ~ ExponentialFamily(\\eta)} $. 即，在给出x和$ \\color{black}{\\theta} $的情况下，y服从某种参数为$ \\color{black}{\\eta} $的指数族分布。

2. 给出一个x值，我们的目标是预测T(y)的值，在大多数情况下，T(y)就是y，这意味着我们所要求出的假设函数$ \\color{black}{h(x)} $为$\\color{black}{h(x) = E[y/x]} $

3. 自然参数 $ \\color{black}{\\eta} $和输出x具有线性关系：$ \\color{black}{\\eta = \\theta\^T x} $

#### 最小二乘模型

下面来阐述最小二乘模型是广义线性模型的的一个特例，模型中需要预测的y值是一个连续的值，而且在给出x时y服从高斯分布，由上面讲述所知，高斯分布也是指数族分布，而且我们有$ \\color{black}{\\mu = \\eta} $，所以由广义线性模型的假设可得：

$$ \\color{black}{\\begin{align}
h\_\\theta(x) \&= E[y|x; \\theta] \\\\
\& = \\mu \\\\
\&= \\theta \\\\
\&= \\theta\^T x
\\end{align} } $$

#### logistic回归

在二分类问题中，y是有两个取值，所以我们用伯努力分布来描述在给出x条件下y的分布。由上一节中得出的结论：$ \\color{black}{\\phi = 1 / (1+e ^{-\\eta})} $，而由于伯努力分布的期望为$ \\color{black}{\\phi} $，代入广义线性模型中，有：

$$ \\color{black}{\\begin{align}
h\_\\theta(x) \&= E[y|x; \\theta] \\\\
\&= \\phi \\\\
\&= 1/\\left(1+e\^{-\\eta}\\right) \\\\
\&= 1/\\left(1+e\^{-\\theta\^T x}\\right)
\\end{align} } $$

这就是为什么在logistic回归中选择sigmoid函数的原因！当假设y在给出x的条件的分布为伯努力分布时，根据广义线性模型推导出的结论就是sigmoid方程.

#### 多项式分布

这里考虑用广义线性模型解决一个实际问题：多分类问题。假设在分类问题中，我们需要的取值不仅仅是0和1，而是一个更多的离散值：$\\color{black}\\{{1, 2, ... k\\}}$。其概率分布为$ \\color{black}{p(y=i) = \\phi\_i} $，由于$ \\color{black}{\\sum \\phi\_i = 1} $，可以只用k-1个参数来表示，最后一个$ \\color{black}{\\phi\_k = 1 - \\sum\_{i=1}\^{k-1} \\phi\_i}$

为了表示为指数族分布，需要定义一些变量的表示。不同于前面的模型，这里的T(y)是一个k-1维的向量：

$$ \\color{black}{
T(1) = \\begin{bmatrix} 
 1 \\\\ 0 \\\\ 0 \\\\ \\vdots \\\\ 0
 \\end{bmatrix}, 
T(2) = \\begin{bmatrix} 
 0 \\\\ 1 \\ 0 \\\\ \\vdots \\\\ 0
 \\end{bmatrix}, 
 \\cdots,
T(k-1) = \\begin{bmatrix} 
 0 \\\\ 0 \\\\ 0 \\\\ \\vdots \\\\ 1
 \\end{bmatrix}, 
T(k) = \\begin{bmatrix} 
 0 \\\\ 0 \\\\ 0 \\\\ \\vdots \\\\ 0
 \\end{bmatrix} 
}$$

这里再引入一个函数：1{arg},如果参数为真时，值为1，参数为假时，值为0：$\\color{black}{1\\{True\\} = 1, 1\\{False\\} = 0}  $。那么T(y)的值可以表示为$ \\color{black}{(T(y))\_i = 1\\{y=i\\}} $，所以就有$ \\color{black}{E[\\left(T(y)\\right)\_i] = P(y=i) = \\phi\_i} $。然后做指数族分布的推导：

$$ \\color{black}{\\begin{align}
p(y;\\phi) \&= \\phi\_1\^{1\\{y=1\\}} \\phi\_2\^{1\\{y=2\\}} \\cdots \\phi\_k\^{1\\{y=k\\}} \\\\
 \&= \\phi\_1\^{1\\{y=1\\}} \\phi\_2\^{1\\{y=2\\}} \\cdots \\phi\_k\^{1-\\begin{matrix}\\sum\_{i=1}\^{k-1} 1\\{y=i\\} \\end{matrix}} \\\\
 \&= \\phi\_1\^{(T(y))\_1} \\phi\_2\^{(T(y))\_2} \\cdots \\phi\_k\^{1-\\begin{matrix}\\sum\_{i=1}\^{k-1} (T(y))\_i \\end{matrix}} \\\\
 \&= exp \\left( \\left(T(y)\\right)\_1 log(\\phi\_1 / \\phi\_k) + \\left(T(y)\\right)\_2 log(\\phi\_2 / \\phi\_k) + \\cdots + \\left(T(y)\\right)\_{k-1} log(\\phi\_{k-1} / \\phi\_k) + log (\\phi\_k) \\right) \\\\
 \&= b(y) exp\\left(\\eta\^T T(y) - a(\\eta) \\right)
\\end{align} } $$

由此，可以得出GLM的各个参数：

$$ \\color{black}{\\begin{align}
\\eta \&= \\left[ \\begin{matrix} log(\\phi\_1 / \\phi\_k) \\\\ log(\\phi\_2 / \\phi\_k) \\\\ \\vdots \\\\ log(\\phi\_{k-1} / \\phi\_k) \\end{matrix} \\right] \\\\
a(\\eta) \&= - log(\\phi\_k) \\\\
b(y) \&= 1 \\\\
\\end{align} } $$

连接函数为：

$$ \\color{black}{\\eta\_i = log \\frac {\\phi\_i} {\\phi\_k}} $$

为了简便，定义$ \\color{black}{\\eta\_k = log(\\phi\_k / \\phi\_k) = 0} $，对连接函数进行变换：

$$ \\color{black}{\\begin{align}
e\^{\\eta\_i} \&= \\frac {\\phi\_i} {\\phi\_k} \\\\
\\phi\_k e\^{\\eta\_i} \&= \\phi\_k \\\\
\\phi\_k \\sum\_{i=1}\^k e\^{\\eta\_i} \&= \\sum\_{i=1}\^k \\phi\_i = 1
\\end{align} } $$

对上式做代换，可得：

$$ \\color{black}{\\phi\_i = \\frac {e\^{\\eta\^i}} {\\sum\_{j=1}\^k e\^{\\eta\_j}}} $$

根据GLM的假设3，可得$ \\color{black}{\\eta\_i = \\theta\_i\^T x (for i = 1,\\cdots,k-1) }$，这里$ \\color{black}{\\theta\_1, \\cdots, \\theta\_{k-1} \\in \\mathbb{R}\^{n+1}}  $是模型的参数。为了方便，我们定义$ \\color{black}{\\theta\_k = 0} $，这样，我们假设在给出x时y的分布为：

$$ \\color{black}{\\begin{align}
p(y=i|x;\\theta) \&= \\phi\_i \\\\
 \&= \\frac {e\^{\\eta\_i}} {\\sum\_{j=1}\^k e\^{\\eta\_j}} \\\\
 \&= \\frac {e\^{\\theta\_i\^T x}} {\\sum\_{j=1}\^k e\^{\\theta\_j\^T x}}
\\end{align} } $$

这个预测多分类的模型叫softmax回归，它是对logistic回归的推广。得到的假设如下：

$$ \\color{black}{\\begin{align}
h\_\\theta(x) \&= E\\left[T(y)|x;\\theta \\right] \\\\
 \&= E\\left[\\left. \\begin{matrix} 1\\{y=1\\} \\\\ 1\\{y=2\\} \\\\ \\vdots \\\\ 1{y=k-1} \\end{matrix} \\right|x;\\theta \\right] \\\\
 \&= \\left[ \\begin{matrix} \\phi\_1 \\\\ \\phi\_2 \\\\ \\vdots \\\\ \\phi\_{k-1} \\end{matrix} \\right] \\\\
 \&= \\left[ \\begin{matrix} \\frac {exp(\\theta\_1\^T x)} {\\sum\_{j=1}\^k exp(\\theta\_j\^T x)}  \\\\ \\frac {exp(\\theta\_2\^T x)} {\\sum\_{j=1}\^k exp(\\theta\_j\^T x)} \\\\ \\vdots \\\\ \\frac {exp(\\theta\_{k-1}\^T x)} {\\sum\_{j=1}\^k exp(\\theta\_j\^T x)} \\end{matrix} \\right]
\\end{align} } $$

用最大似然函数来求解参数，如下：

$$ \\color{black}{ l(\\theta) = \\sum\_{i=1}\^m log p\\left( y\^{(i)}|x\^{(i)}; \\theta \\right) = \\sum\_{i=1}\^m log \\prod\_{l=1}\^k \\left( \\frac {e\^{\\theta\_l\^T x\^{(i)}}} {\\sum\_{j=1}\^k e\^{\\theta\_j\^T x\^{(i)}}} \\right)\^{1\\{y\^{(i)} = l\\}} } $$

接下来，就可以将最大似然函数最大化，然后用梯度下降方法或者牛顿方法来求解参数。


### 参考

- 斯坦福大学公开课--机器学习课程: <http://v.163.com/special/opencourse/machinelearning.html>
- 课程课件下载地址: <http://cimg3.163.com/edu/open/ocw/jiqixuexikecheng.zip>
- logistic模型: <http://zh.wikipedia.org/wiki/Logit%E6%A8%A1%E5%9E%8B>
- Softmax回归: <http://ufldl.stanford.edu/wiki/index.php/Softmax%E5%9B%9E%E5%BD%92>
- 广义线性模型: <http://zh.wikipedia.org/wiki/%E5%BB%A3%E7%BE%A9%E7%B7%9A%E6%80%A7%E6%A8%A1%E5%BC%8F>
