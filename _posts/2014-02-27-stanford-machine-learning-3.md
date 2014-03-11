---
layout: post
title: "机器学习课程总结（三）"
description: "斯坦福大学的机器学习公开课学习笔记。本文主要介绍了生成学习算法, 朴素贝叶斯，拉普拉斯平滑。"
category: machine-learning
tags: [机器学习, 生成学习算法, 朴素贝叶斯, 拉普拉斯平滑, 多元正态分布]
---
{% include JB/setup %}

### 介绍

本文是机器学习课程笔记的第三篇，首先介绍生成学习算法（Generative Learning algorithms）的概念，其中代表性的算法是高斯判别分析（Gaussian discriminant analysis, GDA）；随后介绍了朴素贝叶斯算法和拉普拉斯平滑。

<!-- more -->

### 生成学习算法

之前两节中的学习算法模型都是计算在给出x的条件下y的条件概率分布：$ \\color{black}{p(y|x;\\theta)} $。假如有一个分类问题：我们需要区分大象(y=1)和狗(y=0)，基于动物的一些特征，在之前的算法中，需要找一个超平面去分隔大象和狗，大象和狗落在这个超平面的两侧，预测时需要检查新数据落在平面的哪一侧。

这一节中，介绍一种完全不同的方法。第一，查看大象的特征，为大象建立一个模型来描述大象长什么样子，第二，为狗建立一个模型描述狗长什么样子，第三，当我们需要判别一个新的动物的时候，将这个动物分别应用到两个模型中，看看这个动物更像大象还是更像狗。这种学习方法被称为**生成学习算法**（Generative Learning algorithms）。在前面的模型中，我们直接对$ \\color{black}{p(y|x)} $和$ \\color{black}{p(y)} $进行预测，在生成学习算法中，我们尝试对$ \\color{black}{p(x|y)} $建立模型，如$ \\color{black}{p(x|y)=0} $描述了狗的特征的分布，$ \\color{black}{p(x|y=1)} $描述了大象了特征的分布。

在对$ \\color{black}{p(y)} $和 $ \\color{black}{p(x|y)} $进行建立模型之后，运用贝叶斯定律去生成给出x的情况下y的分布：

$$ \\color{black}{p(y|x) = \\frac {p(x|y)p(y)} {p(x)}} $$

这里，分母可以用以下方法来计算：$ \\color{black}{p(x) = p(x|y=1)p(y=1) + p(x|y=0)p(y=0)} $。事实上，如果需要计算最大的p(y|x)，就不需要计算p(x)的值，因为：

$$ \\color{black}{arg\\,max\\,p(y|x) = arg\\,max\\,\\frac{p(x|y)p(y)}{p(x)} = arg\\,max\\,p(x|y) p(y)} $$

### 高斯判别分析

高斯判别分析（Gaussian discriminant analysis, GBA）是生成学习算法的一个例子，在这个算法中，我们假设$ \\color{black}{p(x|y)} $服从多元正态分布(multivariate normal distribution)。

#### 多元正态分布

多元正态分布是一个n维的分布，它有均值向量：$ \\color{black}{\\mu \\in \\mathbb{R}\^n} $ 和 方差矩阵 $ \\color{black}{\\Sigma \\in \\mathbb{R}\^{n \\times n}} $，其密度函数为：

$$ \\color{black}{p(x;\\mu,\\Sigma) = \\frac{1} {(2\\pi)\^{n/2}\\left|\\Sigma\\right|\^{1/2}} \\exp \\left(-\\frac{1}{2}(x-\\mu)\^T \\Sigma\^{-1}(x-\\mu)\\right)} $$

上式中，$ \\color{black}{|\\Sigma|} $ 表示行列式。

假设一个随机变量X服从$ \\color{black}{N(\\mu, \\Sigma)} $，它的均值是：

$$ \\color{black}{E[X] = \\int\_x xp(x;\\mu,\\Sigma)dx = \\mu} $$

方差是：

$$ \\color{black}{Cov(X) = \\Sigma} $$

下面是一些多元正态分布的示例图：

![高斯分布示例图](/assets/images/machine-learing/ml3-1.jpg)

最左边的图表示一个标准正态分布，其均值为0向量，方差为单位矩阵（$  \\color{black}{\\Sigma = I} $），中间的一幅图表示了均值为0向量，方差为 0.6I 的分布，最右边的一幅图均值为0向量，方差为 2I 的分布。可以看出，方差越大，分布越扁平，方差越小，分布越尖锐。

![高斯分布示例图](/assets/images/machine-learing/ml3-2.jpg)

上面三幅图的均值向量都为0向量，方差矩阵分别为：

$$ \\color{black}{\\Sigma = \\left[ \\begin{matrix} 1 \\ 0 \\\\ 0 \\ 1 \\end{matrix}\\right];
\\Sigma = \\left[ \\begin{matrix} 1 \\ 0.5 \\\\ 0.5 \\ 1 \\end{matrix}\\right];
\\Sigma = \\left[ \\begin{matrix} 1 \\ 0.8 \\\\ 0.8 \\ 1 \\end{matrix}\\right]
} $$

可以看出，当随着副对角线的元素的增大，其图像趋近于45度方向扁平（如果副对角线的元素为负数，为135度方向），从下面图像的角度可以看的更清楚：

![高斯分布示例图](/assets/images/machine-learing/ml3-3.jpg)

当固定方差矩阵为$ \\color{black}{\\Sigma = I} $，变化$ \\color{black}{\\mu} $时，可以看出分布图会在平面上移动：

![高斯分布示例图](/assets/images/machine-learing/ml3-4.jpg)

上面三幅图的均值向量分别为：

$$ \\color{black}{\\mu = \\left[ \\begin{matrix} 1 \\\\ 0 \\end{matrix}\\right];
\\mu = \\left[ \\begin{matrix} -0.5 \\\\ 0 \\end{matrix}\\right];
\\mu = \\left[ \\begin{matrix} -1 \\ \\\\ -1.5 \\end{matrix}\\right]
} $$

#### 高斯判别分析模型

当在分类问题中，输入的特征变量x是一个连续的随机变量时，我们就可以使用GDA模型，假设p(x|y)服从多元正态分布，这个模型为：

$$ \\color{black}{\\begin{align}
y \&\\sim Bernoulli(\\phi) \\\\
x|y=0 \&\\sim N(\\mu\_0, \\Sigma) \\\\
x|y=1 \&\\sim N(\\mu\_1, \\Sigma) \\\\
\\end{align}} $$

写出分布函数：

$$ \\color{black}{\\begin{align}
p(y) \&= \\phi\^y (1-\\phi)\^{1-y} \\\\
p(x|y=0) \&= \\frac {1}{(2\\pi)\^{n/2}\\left|\\Sigma\\right|\^{1/2}} \\exp \\left(-\\frac {1}{2} (x-\\mu\_0)\^T \\Sigma\^{-1}(x-\\mu\_0)\\right)\\\\
p(x|y=1) \&= \\frac {1}{(2\\pi)\^{n/2}\\left|\\Sigma\\right|\^{1/2}} \\exp \\left(-\\frac {1}{2} (x-\\mu\_1)\^T \\Sigma\^{-1}(x-\\mu\_1)\\right)\\\\
\\end{align}} $$

这里，两个高斯分布使用的是相同的方差，不同的均值，模型的参数有$ \\color{black}{\\phi, \\Sigma, \\mu\_0, \\mu\_1} $，其似然函数如下：

$$ \\color{black}{\\begin{align}
l(\\phi,\\mu\_0,\\mu\_1,\\Sigma) \&= \\log \\prod\_{i=1}\^m p\\left(x\^{(i)},y\^{(i)};\\phi,\\mu\_0,\\mu\_1,\\Sigma,\\right) \\\\
 \&= \\log \\prod\_{i=1}\^m p\\left(x\^{(i)} | y\^{(i)};\\mu\_0,\\mu\_1,\\Sigma\\right) p(y\^{(i)};\\phi)
\\end{align}} $$

求解过程忽略，最大似然函数的解为：

$$ \\color{black}{\\begin{align}
\\phi \&= \\frac{1}{m}\\sum\_{i=1}\^m 1\\{y\^{(i)} = 1\\} \\\\
\\mu\_0 \&= \\frac{\\sum\_{i=1}\^m 1\\{y\^{(i)} = 0\\}x\^{(i)}}{\\sum\_{i=1}\^m 1\\{y\^{(i)} = 0\\}} \\\\
\\mu\_1 \&= \\frac{\\sum\_{i=1}\^m 1\\{y\^{(i)} = 1\\}x\^{(i)}}{\\sum\_{i=1}\^m 1\\{y\^{(i)} = 1\\}} \\\\
\\Sigma \&= \\frac{1}{m}\\sum\_{i=1}\^m \\left(x\^{(i)}-\\mu\_{y\^{(i)}}\\right) \\left(x\^{(i)}-\\mu\_{y\^{(i)}}\\right)\^T
\\end{align}} $$

![高斯判别分析模型示例](/assets/images/machine-learing/ml3-5.jpg)

求出的模型包含两个多元分布模型，图片如上图所示，由于两个模型的方差矩阵相同，因此图形的形状一致，而两个模型的均值向量不同，决定了两个模型的位置不同，分别表示了正样本和负样本的分布。

#### 高斯判别分析和logistic回归

高斯判别分析和logistic回归模型有一些有趣的联系。如果我们把$ \\color{black}{p(y=1|x;\\phi,\\mu\_0,\\mu\_1,\\Sigma)} $看成是x的函数，那么这个式子可以被表示为：

$$ \\color{black}{
p(y=1|x;\\phi,\\mu\_0,\\mu\_1,\\Sigma) = \\frac{1}{1+\\exp(-\\theta\^T x)}
} $$

这里$ \\color{black}{\\theta} $是$ \\color{black}{\\phi,\\mu\_0,\\mu\_1,\\Sigma} $的函数，可以看出上式是logistic回归模型的函数。那么GDA模型和logistic模型之间有什么关系呢？

如果p(x|y)是一个多元高斯分布，那么p(y|x)一定服从logistic分布，反之没有这个关系。GDA模型比logistic模型做了更强的假设，意味着如果对p(x|y)的假设如果是正确的，那么GDA模型表现更好。对于较大的训练集来说，GDA几乎是最好的模型。在训练集较小时，GDA也有着不俗的表现，因为GDA是一个对数据不敏感的模型（在很小的训练集中就可以表现很好）。

相反，logistic模型做了比较弱的假设，意味着它有更强的鲁棒性，并且对模型的假设不敏感。有更多的假设可以导致p(y|x)服从logistic函数，例如p(x|y)服从波松分布，那么p(y|x)也是logistic函数。但是如果用GDA算法应用在了非高斯分布的数据中，那么GDA将会有很差的表现。

### 朴素贝叶斯

一个现实中的例子：垃圾邮件过滤。要做的是区分一个邮件是否是一个广告邮件，输入是邮件的内容。垃圾邮件过滤是**文本分类**的一个例子。这里有一批用于训练的数据：一批邮件，被人工标明是否是垃圾邮件。

现在用一个特征向量来表示一封电邮的特征，首先要构造一个字典，如果邮件包含字典中第i个单词，则$ \\color{black}{x\_i} $为1，否则$ \\color{black}{x\_i} $为0。特征向量的维度和字典的长度相等。

在构造了特征向量之后，我们来构造一个判别学习模型，那么就要建模p(x|y), 但是通常情况下字典的长度都很大，如果字典中有50000个单词，那么多项式分布将会有$ \\color{black}{2\^{50000}} $种可能的输出，显然太大了。在这种情况下，我们需要做一个很强的假设，假设$ \\color{black}{x\_i} $在给出y的条件下是独立的，这个假设被称为**朴素贝叶斯假设(Naive Bayes assumption)**，而我们得到的分类算法被称为**朴素贝叶斯分类**。例如，如果某个邮件的y=1，那么邮件是$ \\color{black}{x\_{1287} = 1} $和$ \\color{black}{x\_{2342} = 1} $之间是独立的（即在给出y的条件下，$ \\color{black}{x\_{1287}}$和$ \\color{black}{x\_{2342}} $是条件独立的}）。

在这个假设下有：

$$ \\color{black}{\\begin{align}
p(x\_1, ... , x\_{50000}|y) \&= p(x\_1|y)p(x\_2|y, x1)\\cdots p(x\_{50000}|y, x\_1,...,x\_{49999}) \\\\
 \&= p(x\_1|y)p(x\_2|y)\\cdots p(x\_{50000}|y) \\\\
 \&= \\prod\_{i=1}\^{n} p(x\_i|y)
\\end{align}} $$

和往常一样，对于训练集$ \\color{black}{\\left(x\^{(i)}, y\^{(i)}\\right); i=1,...,m} $, 写出上式的似然函数：

$$ \\color{black}{L(\\phi\_y, \\phi\_{i|y=0}, \\phi\_{i|y=1})} $$

给出各参数的似然估计：

$$ \\color{black}{\\begin{align}
\\phi\_{j|y=1} \&= \\frac {\\sum\_{i=1}\^m 1\\{x\_j\^{(i)}=1 \\land y\^{(i)} = 1\\}} {\\sum\_{i=1}\^m 1 \\{y\^{(i)} = 1\\}} \\\\
\\phi\_{j|y=0} \&= \\frac {\\sum\_{i=1}\^m 1\\{x\_j\^{(i)}=1 \\land y\^{(i)} = 0\\}} {\\sum\_{i=1}\^m 1 \\{y\^{(i)} = 0\\}} \\\\
\\phi\_y \&= \\frac {\\sum\_{i=1}\^m 1\\{y\^{(i)} = 1\\}} {m} \\\\
\\end{align}} $$

得到这些参数之后，用以下的公式计算一封新的邮件是否是垃圾邮件。

$$ \\color{black}{\\begin{align}
p(y=1|x) \&= \\frac {p(x|y=1)p(y=1)} {p(x)} \\\\
 \&= \\frac {\\left(\\prod\_{i=1}\^n p(x\_i|y=1)\\right)p(y=1)} {\\left(\\prod\_{i=1}\^n p(x\_i|y=1)\\right)p(y=1) + \\left(\\prod\_{i=1}\^n p(x\_i|y=0)\\right)p(y=0)}
\\end{align}} $$

#### 拉普拉斯平滑

上面描述的朴素贝叶斯算法在很多情况下可以达到很好的效果，有一些简单的改动能使它工作的更好，特别是文本分类，如拉普拉斯平滑。

拉普拉斯平滑是为了解决以下问题：当采用朴素贝叶斯对训练集建立了一个分类模型后，来了一封新的邮件是NIPS会议的邮件，其中有一个单词"nips"，但是这是收到的第一封NIPS会议的邮件，之前的训练集中从来没有出现过"nips"这个单词。假设"nips"这个单词是字典中的第35000个单词，模型中相关的参数就为：

$$ \\color{black}{\\begin{align}
\\phi\_{35000|y=1} \& = \\frac {\\sum\_{i=1}\^m 1\\{x\_{35000}\^{(i)}=1 \\land y\^{(i)} = 1\\}}{\\sum\_{i=1}\^m 1 \\{y\^{(i)} = 1\\}} = 0 \\\\
\\phi\_{35000|y=0} \& = \\frac {\\sum\_{i=1}\^m 1\\{x\_{35000}\^{(i)}=1 \\land y\^{(i)} = 0\\}}{\\sum\_{i=1}\^m 1 \\{y\^{(i)} = 0\\}} = 0 \\\\
\\end{align}} $$

因为"nips"这个单词从来没有出现过，所以得到的参数都为0. 从而，得到这封邮件的概率值：

$$ \\color{black}{\\begin{align}
p(y=1|x) \&= \\frac {\\left(\\prod\_{i=1}\^n p(x\_i|y=1)\\right)p(y=1)} {\\left(\\prod\_{i=1}\^n p(x\_i|y=1)\\right)p(y=1) + \\left(\\prod\_{i=1}\^n p(x\_i|y=0)\\right)p(y=0)} \\\\
 \&= \\frac {0}{0} 
\\end{align}} $$

这将会使无法对这封邮件进行预测。出现这种结果的原因是我们做了不准确的假设：某个单词没有在训练集中出现过，我们就假设它出现的概率为0. 怎么解决呢？对于一个随机变量z，它的范围是$ \\color{black}{\\{1,2,3,\\ldots,k\\}} $，多项式参数是$ \\color{black}{\\phi\_i = p(z=i)} $经过m次试验后的预测结果$ \\color{black}{\\{z\^{(1)},z\^{(2)},z\^{(3)},\\ldots,z\^{(m)}\\}} $，概率的极大似然估计为

$$\\color{black}{\\phi\_j = \\frac{\\sum\_{i=1}\^m 1\\{z\^{(i)}=j\\}} {m}} $$

这个公式中，遇到上面的问题时，可能会得到0. 这里我们应用**拉普拉斯平滑**来解决。

$$\\color{black}{\\phi\_j = \\frac{\\sum\_{i=1}\^m 1\\{z\^{(i)}=j\\} + 1} {m + k}} $$

即在分子上加1，在分母加k（z的取值范围的大小），增加这个值后，$ \\color{black}{\\sum\_{i=1}\^m \\phi_i} $的值还是1.

在朴素贝叶斯算法中，应用拉普拉斯平滑：

$$ \\color{black}{\\begin{align}
\\phi\_{j|y=1} \&= \\frac {\\sum\_{i=1}\^m 1\\{x\_j\^{(i)}=1 \\land y\^{(i)} = 1\\} + 1} {\\sum\_{i=1}\^m 1 \\{y\^{(i)} = 1\\} + 2} \\\\
\\phi\_{j|y=0} \&= \\frac {\\sum\_{i=1}\^m 1\\{x\_j\^{(i)}=1 \\land y\^{(i)} = 0\\} + 1} {\\sum\_{i=1}\^m 1 \\{y\^{(i)} = 0\\} + 2} \\\\
\\end{align}} $$

### 参考

1. 斯坦福大学公开课--机器学习课程: <http://v.163.com/special/opencourse/machinelearning.html>
2. 课程课件下载地址: <http://cimg3.163.com/edu/open/ocw/jiqixuexikecheng.zip>
3. 朴素贝叶斯分类器: <http://zh.wikipedia.org/zh-cn/%E6%9C%B4%E7%B4%A0%E8%B4%9D%E5%8F%B6%E6%96%AF%E5%88%86%E7%B1%BB%E5%99%A8>
