---
layout: post
title: "机器学习课程总结（六）"
description: "机器学习课程笔记第6篇，介绍机器学习的学习理论，从一个假设集合中选取最好的模型。"
category: machine-learning
tags: [机器学习, 偏差, 方差, 经验风险最小化, 联合界, 一致收敛性, VC维]
comments: true
---

## 简介

本篇对应着公开课的[第9讲][course9]。从本篇开始，课程从另一个角度来诠释机器学习的概念——学习理论。主要内容有有**偏差和方差的权衡**，**经验风险最小化**，**联合界**，**一致收敛性**，**VC维**。

## 偏差和方差的权衡

对于**欠拟合**和**过拟合**的概念，在本系列课程总结的[第一篇][mlpost1]中就有过介绍，相信本篇文章的读者对这两个概念也不会陌生。这里要讲的**偏差**和**方差**其实就对应着欠拟合和过拟合。

用线性回归来举例，看如下的示例:

![偏差/方差示例]({{ site.ASSET_PATH }}/images/machine-learing/ml6-1.jpg)

最右边的图形对训练数据的一个5元的模型，它很好的反映了训练数据的每个点，但是并不是一个很好的模型，因为它很可能不能代表真实数据中输入和输出的关系。这里提出一个**泛化误差**的概念，它是一个假设的测试集(非训练集)的期望误差。

在上面的示例中，最左边和最右边的模型均有很大的泛化误差，然而，这两个模型造成误差的原因却是相反的。讨论左边的模型，由于$$ x $$ 和 $$ y $$的关系并不是线性的，用一个线性模型去拟合时，即使训练数据再多也不能很好的得到数据的结构。这里定义**偏差**为拟合一个足够大的训练集时的泛化误差。所以，对于左边的模型来说，具有一个较大的偏差，对应着欠拟合。

第二个讨论的泛化误差，是由模型拟合过程中的**方差**构成。对于右边的模型来说，很好匹配的训练模型，但是很可能在实际数据中取得错误的结果。这里称这个模型也具有较大的**方差**，对应着过拟合。

通常，我们要在偏差和方差之间做一个权衡。如果我们的模型很“简单”并且参数的个数很少，那么很有可能具有高偏差（低方差）；如果模型很“复杂”并且参数个数非常多，那么很有可能具有高方差（低偏差）。

## 经验方差最小化

这里要对学习理论做一个了解，首先需要去探索以下几个问题：第一，是否能够对方差/偏差权衡进行形式化的表示？例如，可以决定几元模型能更好的符合训练数据。第二，我们是否能够将泛化误差和模型在训练集上的误差关联起来？第三，是否有一个情况来表示我们的算法的效果较好。

这里先介绍两个简单但是非常有用的引理。

**联合界引理** 假设$$ A_1, A_2, \ldots, A_k $$是不同的事件（可能不独立），则一定有：

$$ P(A_1 \cup \cdots \cup A_k) \le P(A_1) + \ldots + P(A_k) $$

联合界引理在概率论中是一条公理，这里不去证明它。但是很好理解，k个事件中任何一个发生的概率不大于k个事件发生的概率之和。

**Hoeffding不等式** 假设$$ Z_1, \ldots, Z_m $$ 是独立同分布（independent and identically distributed, iid）的随机变量，服从伯努力分布，$$ P(Z_i=1)=\phi , P(Z_i = 0) = 1 - \phi $$，设$$ \hat{\phi} = (1 / m) \sum_{i=1}^m Z_i $$为这些随机变量的期望，设$$ \gamma > 0 $$ 是一个固定值，那么有：

$$ P(\vert \phi - \hat{\phi} \vert > \gamma ) \le 2 \exp ( - 2 \gamma^2 m) $$

这个引理说明，假如我们以$$ \hat{\phi} $$表示m个服从伯努力分布的随机变量的平均值，用来估计真实的$$ \phi $$值，那么m越大，$$ \hat{\phi} $$和$$ \phi $$值的差距就越小。

使用这两个引理，就可以证明学习理论中一些更深层次和更重要的结论。

这里以二分类问题来做研究，假设分类结果为$$ y \in \{ 0, 1 \} $$，训练集为$$ S = \{ (x^{(i)}, y^{(i)}); i=1,\ldots,m \} $$，并且$$ (x^{(i)}, y^{(i)}) $$服从某种概率分布 $$ \mathcal{D} $$。现在定义**训练误差**(在学习理论中被称为**经验风险**或者**经验误差**)：

$$ \hat{\epsilon} (h) = \frac {1} {m} \sum_1^m 1 \{ h((x^{(i)}) \neq y^{(i)} \} $$

这个值就是$$ h $$ 误分的概率。再定义泛化误差：

$$ \epsilon (h) = P_{(x,y) \sim \mathcal{D}} (h(x) \neq y)$$

泛化误差可以用来衡量假设效果。

对于线性分类来说，$$ h_\theta (x) = 1 \{ \theta^T x \ge 0 \} $$，拟合$$ \theta $$最合理的方法是什么？使训练误差最小：

$$ \hat{\theta} = \arg \min_\theta \hat{\epsilon} (h_\theta) $$

这个过程称为**经验风险最小化**(empirical risk minimization, ERM)，并且所求的假设为$$\hat{h} = h_{\hat{\theta}}$$. ERM是最基本的学习算法。目的是使训练误差最小化，这个非凸性优化问题实际是一个NP难的问题，logistic回归算法可以看成是这个算法的近似。

现在重新定义ERM为另一种等价的形式，对于我们的算法目标，相对于以上是选取一个合适的参数，现在要使目标变为选取一个合适的函数，假设函数集：$$ \mathcal{H} = \{ h_\theta, \theta \in \mathbb{R}^{n+1} \}$$ 是所有可能的线性分类器。

ERM现在就变成一个从$$ \mathcal{H} $$中选取使训练误差最小化的假设，那么就得到了对假设的估计：

$$ \hat{h} = \arg \min_{h \in \mathcal{H}} \hat{\epsilon} (h) $$

## 有限的假设集

现在我们针对一个有限的假设集合做讨论$$ \mathcal{H} = \{h_1, \ldots, h_k\} $$，即$$ \mathcal{H} $$是一个k个从$$\mathcal{X}$$到$$\{0, 1\}$$的映射函数，那么经验方差最小化的结论$$\hat{h}$$就是这k个函数是训练误差最小的一个假设。

我们求解$$\hat{h}$$的策略分为两部分，第一步，证明$$ \hat{\epsilon}(h)$$是对$$ \epsilon (h)$$的一个可靠的估计，第二步，证明$$\hat{h}$$的泛化误差有一个上界。

### 一致收敛定理

现在先证明第一步，假设随机变量Z服从伯努力分布，然后假设$$ Z=1\{h_i(x) \neq y\} $$,所以，Z代表了分类器误判的概率。类似的，假设$$ Z_j = 1 \{ h_i (x^{(j)}) \neq y^{(j)}\} $$，这里可以看到误判概率为一个随机变量$$\epsilon(h)$$，等于Z的期望值，所以训练误差：

$$ \hat{\epsilon}(h_i) = \frac{1}{m} \sum_{j=1}^m Z_j $$

所以 $$\hat{\epsilon}(h_i)$$ 是m个随机变量$$Z_j$$的平均值，并且这些随机变量服从一个均值为$$\epsilon (h_i)$$的伯努力分布，应用Hoeffding不等式，可以得到：

$$P(\vert \epsilon(h_i) - \hat{\epsilon} (h_i) \vert > \gamma ) \le 2 \exp (-2 \gamma^2 m))$$

上式表明，对于特定的假设$$h_i$$，训练误差等于泛化误差的概率很大，但是我们要证明的是对于所有假设，这一结论都成立。现在用$$A_i$$表示$$\vert \epsilon(h_i) - \hat{\epsilon}(h_i) \vert > \gamma $$，所以对于任何特定的$$A_i$$，有$$P(A_i) \le 2 \exp (-2 \gamma^2 m))$$，使用联合界引理，可以得到

$$\begin{align*}
P(\exists h \in \mathcal{H}. \vert \epsilon(h_i) - \hat{\epsilon(h_i)} \vert > \gamma) & = P (A_1 \cup \cdots \cup A_k) \\
 & \le \sum_{i=1}^k P(A_i) \\
 & \le \sum_{i=1}^k 2 \exp(-2 \gamma^2 m) \\
 & = 2k \exp(-2 \gamma^2 m)
\end{align*}$$

同时用1减去两边，得到

$$\begin{align*}
P (\lnot \exists h \in \mathcal{H}. \vert \epsilon(h_i) - \hat{\epsilon(h_i)} \vert > \gamma ) &= P(\forall h \in \mathcal{H}. \vert \epsilon(h_i) - \hat{\epsilon(h_i)} \vert \le \gamma) \\
& \ge 1 - 2k \exp(-2 \gamma^2 m)
\end{align*}$$

上式表明，至少有$$1 - 2k \exp(-2 \gamma^2 m)$$的概率，使得所有的假设，其泛化误差和训练误差的差值都在$$\gamma$$的范围内。这个式子称为**一致收敛定理**。

### 固定参数$$m, \gamma$$

在上面的讨论中，有三个参数$$m, \gamma$$和误差的概率，这三个参数是关联的，我们可以固定其中两个，来推导第三个。例如，给出$$\gamma$$和$$\delta > 0$$，求出m至少多在才能保证可信度至少为$$1 - \delta$$，并使得泛化误差的错误率在训练误差的错误率的$$\gamma$$范围内？

现在设$$\delta = 2k \exp(-2 \gamma^2 m) $$，然后求解出m，可得

$$ m \ge \frac{1} {2 \gamma^2} \log \frac {2k}{\delta}$$

这个结论说明了，为了使算法的表现达到某个水平，至少需要多少个训练样本。这也被称为算法的**样本复杂度**。

### 固定参数$$m, \delta$$

现在固定参数$$m, \delta$$来求解$$\gamma$$，为了使可信度为$$1 - \delta$$，对于所有的$$h \in \mathcal{H}$$

$$ \vert \hat{\epsilon(h)} - \epsilon(h) \vert \le \sqrt {\frac{1}{2m} \log \frac {2k}{\delta}} $$

现在假设一致收敛成立的情况下，我们来证明通过 ERM 得到的假设$$\hat{h} = \arg \min_{h \in \mathcal{H}} \hat{\epsilon} (h)$$会有什么样的泛化能力。

假设$$h^* = \arg \min_{h \in \mathcal{H}} \epsilon (h)$$是假设集中最好的一个假设（泛化误差最小的假设）。

$$ \begin{align*} 

\epsilon(\hat{h}) & \le \hat{\epsilon} (\hat{h}) + \gamma \\
 & \le \hat{\epsilon} (h^*) + \gamma \\
 & \le \epsilon(h^*) + 2 \gamma
\end{align*} $$

上式第一行使用了$$ \vert \epsilon(\hat{h}) - \hat{\epsilon} (\hat{h}) \vert \le \gamma $$(一致收敛定理)，第二行由于$$\hat{h}$$是训练误差最小的假设，当然有$$ \hat{\epsilon} (\hat{h}) \le \hat{\epsilon} (h^*) $$，第三行再次应用一致收敛定理，即$$\hat{\epsilon} (h^*) \le \epsilon(h^*) +  \gamma $$，这表明，在一致收敛成立的时候，通过ERM得到的训练误差最小的假设$$\hat{h}$$的泛化误差比假设集中最好的假设的泛化误差差最多$$2 \gamma$$。

再做一些推论可得：

**定理** 令$$\vert \mathcal {H} \vert = k$$，令$$m, \delta$$固定，可信度为$$1 - \delta$$，那么有下式成立：

$$ \epsilon(\hat{h}) \le \left( \min_{h \in \mathcal{H}} \epsilon(h) \right) + 2 \sqrt {\frac{1}{2m} \log \frac {2k}{\delta}} $$

该定理使用了上面的结论来证明，不再详述。该定理反映了开始时讲的方差和偏差的权衡，例如，如果有一个假设集$$\mathcal{H}$$，现在需要换到另一个更大的集合$$\mathcal{H}^{'} \supseteq \mathcal{H} $$，那么第一个因子$$\min_h \epsilon(h)$$就会变小，因为是从一个更大的集合中挑选最小值。所以，换到大集合中后“偏差”会变小，然而，由于k的增加，导致第二个根号因子会变大，这个增大对应着“方差”的增加。

**推论** 令 $$\vert \mathcal{H} \vert = k$$，并且$$\delta, \gamma$$固定，那么至少有$$1-\delta$$的可信度使$$\epsilon(\hat{h}) \le min_{h \in \mathcal{H}} \epsilon(h) + 2\gamma$$成立的前提是：

$$ \begin{align*}
m & \ge \frac{1}{2 \gamma^2} \log \frac{2k}{\delta} \\
 & = O\left( \frac{1}{\gamma^2} \log \frac{k}{\delta} \right) \\
\end{align*}$$

## 无限的假设集

上面对有限的假设集进行了讨论，现在我们设想假设集是无限的，对一个模型$$\mathcal{H}$$来说，假设它有d个参数，虽然理论上这d个参数的取值组合会有无穷多，但是在计算机的表达中，假设每个参数都是64位浮点数表示，那么共存在64d位，所以这个假设集共有$$2^{64d}$$大小。

根据上一节的结论，为了保证$$\epsilon(\hat{h}) \le \epsilon(h^*) + 2 \gamma$$，并且有$$ 1 - \delta$$的可信度，必须满足$$ m \ge O\left( \frac{1}{\gamma^2} \log \frac{64d}{\delta} \right) = O \left(\frac{d}{\gamma^2} \log \frac{1}{\delta}\right)  = O_{\gamma,\delta} (d)$$。所以，需要的训练样本的数目和参数的个数至多是线性相关的。

但是上面的说法是有缺陷的，例如我们写一个线性分类器$$ h_\theta (x) = 1 \{\theta_0 + \theta_1 x_1 + \cdots + \theta_n x_n \ge 0\} $$，这个式子中有n+1个参数，但是上式也可以转成另一种形式$$ h_{u,v} (x) = 1 \{ (u_0^2 - v_0^2) + (u_1^2 - v_1^2) x_1 + \cdots + (u_n^2 - v_n^2) x_n \ge 0\} $$，这个式子中则有2n+2个参数。

为了推导出更合适的参数，定义一些表示方法。

给出一个点集合$$ S = \{x^{(i)}, \ldots, x^{(d)}\}$$，其中$$ x^{(i)} \in \mathcal{X}$$，如果一个假设集$$\mathcal{H}$$ 能够实现 $$ S $$ 中的任意一种标记方式，我们说$$ \mathcal{H} $$可以**分散**$$ S $$.

给出一个假设集$$ \mathcal{H} $$，我们定义它的VC维(Vapnik-Chervonenkis dimension)，写出$$ VC(H) $$，等于$$ \mathcal{H} $$可以分散的最大集合的大小。结果可以是无穷大。

**示例** 那么所有二维线性分类器组成的假设集$$\mathcal{H}$$的VC维是多少呢？看下面的点集合：

![3个点集合]({{ site.ASSET_PATH }}/images/machine-learing/ml6-2.jpg)

这3个点的所有标记方式如下：

![3个点集合所有标记方式]({{ site.ASSET_PATH }}/images/machine-learing/ml6-3.jpg)

可以看到，对于任意一种标记方式来说，二维线性分类器都可以对其进行分类。然而，没有4个点组成的点集合可以被这个假设集分散，所以$$ VC(\mathcal{H}) = 3 $$。

然而，也有一些由3个点组成的集合不能被二维线性分类器打散，如：

![3个点集合的标记方式]({{ site.ASSET_PATH }}/images/machine-learing/ml6-4.jpg)

但是，要证明一个假设的$$ VC(\mathcal{H}) = d $$，只需要一个d大小的点集合能被$$ \mathcal{H}$$分散即可。

更一般的，给出一个n维线性分类器的集合，它的VC维是n+1。

**定理** 给定一个假设集$$ \mathcal{H} $$，令$$d=VC(\mathcal{H})$$，那么至少有$$ 1-\delta $$的概率，对于所有的$$ h \in \mathcal{H}$$，有

$$\vert \epsilon(h) - \hat{\epsilon} (h) \vert \le O(\sqrt{\frac{d}{m} \log \frac{m}{d} + \frac{1}{m} \log \frac{1}{\delta}})$$

那么，至少在$$ 1-\delta $$的概率下，有

$$ \epsilon(\hat{h}) \le \epsilon(h^*) + O(\sqrt{\frac{d}{m} \log \frac{m}{d} + \frac{1}{m} \log \frac{1}{\delta}}) $$

换句话说，如果一个假设集有一个有限的VC维时，一致收敛的发生需要m足够大。根据这个公式可以给出$$\epsilon(h)$$和$$\epsilon(h^*)$$之间关系的一个边界：

**引理** 要保证$$ \vert \epsilon(h) - \hat{\epsilon}(h) \vert \le \gamma$$对于所有的$$ h \in \mathcal{H}$$ 在概率$$ 1 - \delta$$下成立，需要满足$$m = O_{\gamma, \delta}(d)$$.

换句话说，要使模型“较好”的训练，需要的样本数要和假设集的VC维线性相关。而在大多数情况下，VC维的大小和模型的参数是线性相关的，所以，所需的样本数就大体上和模型的参数个数是线性相关的。

[course9]: http://v.163.com/movie/2008/1/F/H/M6SGF6VB4_M6SGJV3FH.html
[mlpost1]: {% post_url 2014-02-11-stanford-machine-learning-1 %}

