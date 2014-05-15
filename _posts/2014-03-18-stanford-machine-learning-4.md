---
layout: post
title: "机器学习课程总结（四）"
description: "机器学习课程笔记第4篇，总结SVM算法的相关概念和理论。"
category: machine-learning
tags: [机器学习, SVM, 支持向量机, 对偶问题, 函数间隔, 几何间隔]
comments: true
---

### 介绍

机器学习课程笔记的第四篇，讲解支持向量机(Support Vector Machine, SVM)学习算法，SVM是最好的监督学习算法之一。在介绍支持向量机之前，首先介绍**间隔**的概念和把数据用一个巨大的间隔分开，这是SVM存在的基础理论；然后，引入**拉格朗日对偶**的概念，深入理解SVM的求解过程。最后，介绍最优间隔分类，这里利用拉格朗日对偶来求解。

<!-- more -->

### 函数间隔和几何间隔

在介绍间隔之前，先了解什么是预测的可信度。在logistic回归中，用$$h\_\theta(x) = g(\theta^T x) $$来预测$$ p(y=1\|x;\theta) $$的值，当$$ \theta^T x \ge 0 $$时，预测的结果为1. 但是看训练集中的数据，$$ \theta^T x $$越大，$$ h\_\theta(x) $$的值也越大，此时也就说明该样本标为1具有更高的可信度。同样，当预测时$$ \theta^T x \gg 0 $$，说明该输入被预测为1也具有更高的可信度，同理当预测时$$ \theta^T x \ll 0 $$，说明该输入被预测为0具有更高的可信度。**可信度**的概念是一个很好的优化目标，我们可以使正样本和负样本在模型中都尽量有更高的可信度，那么模型的效果将会更好，我们将用函数间隔来描述这一问题。

对于另一个不同的问题，考虑如下的图形：

![函数间隔](/assets/images/machine-learing/ml4-1.jpg)

在图中，叉号表示正样本，圆圈表示负样本，判定边界($ \theta^T x = 0 $)被称为分隔超平面。另外，在图中可以看到3个点A，B和C。A点离判定边界最远，可以直观上认为在A点标为1非常可信，C点与判定边界非常接近，直观上可以认为C点被判为1的可信度较低（因为几乎都要分到判定边界另一边了）。在这里我们可以用样本点和判定边界的距离来表示样本的可信度，后续将用几何间隔来描述这一问题。


在SVM模型中，为了方便表示，修改之前的表示符号及公式。用$ \color{black}{y \in \{-1, 1\}} $来表示分类的取值，用参数w，b来表示分类的模型参数：

$$ \color{black}{h_{w,b}(x) = g(w^Tx + b)} $$

其中

$$ \color{black}{g(z) = \begin{cases}
1, if z	\ge 0 \\
-1, otherwise
\end{cases}} $$

#### 函数间隔

给出一个训练样本$ \color{black}{(x^{(i)}, y^{(i)})} $，定义函数间隔是(w,b)的函数：

$$ \color{black}{\widehat{\gamma}^{(i)} = y^{(i)} (w^T x + b)} $$

研究上式，当$ \color{black}{y^{(i)} = 1} $时，为了使函数间隔保持一个较大值，需要$ \color{black}{w^T x + b} $是一个大的正数；相反的，如果$ \color{black}{y^{(i)} = -1} $时，需要$ \color{black}{w^T x + b} $是一个大的负数。在预测的时候，如果$ \color{black}{y^{(i)}(w^T x + b) > 0} $，那么这个预测就是一个正确的预测。由此可见，一个大的函数间隔代表一个可信的正确的预测。

对一个训练集$ \color{black}{S = \{(x^{(i)}, y^{(i)}); i=1,...,m \} }$，定义参数为 (w, b) 的函数间隔为各样本值中的函数间隔的最小值：

$$ \color{black}{\widehat{\gamma} = \min_{i=1,...,m} \widehat{\gamma}^{(i)}} $$

然而，对于线性分类来说，函数间隔并不能很好的描述可信度，原因是模型具有以下特性：模型中，把参数 w 和 b 乘以2，变为 2w 和 2b，并不会改变模型$ \color{black}{h\_{w,b}(x)} $，但是函数间隔的值会加倍，这意味着模型固定的情况下，函数间隔的值并不是固定的。

#### 几何间隔

接下来，我们来讨论**几何间隔**。

![几何间隔](/assets/images/machine-learing/ml4-2.jpg)

以 (w, b) 为参数判定边界的图形如上所示，w 是一个垂直于判定边界的向量，考虑点A，它代表训练样本$ \color{black}{(x^{(i)}, y^{(i)} = 1)} $，点A到判定边界的距离就叫几何间隔。

怎么去计算几何间隔的值呢？$ \color{black}{w/\|\|w\|\|} $是和向量 $ \color{black}{w} $方向相同的单位向量，A点表示为$ \color{black}{x^{(i)}} $，可以推出上图中的B点表示为$ \color{black}{x^{(i)} - \gamma^{(i)} \cdot w/\|\|w\|\| } $，由于所有在判定边界上的点都满足$ \color{black}{w^T x + b = 0} $，所以:

$$ w^T \left( x^{(i)} - \gamma^{(i)} \cdot \frac {w}{\|w\|} \right) + b = 0 $$

可以求出几何间隔：

$$ \gamma^{(i)} = \left( \frac {w}{\|w\|} \right)^T x^{(i)} + \frac {b} {\|w\|} $$

上式为正样本的几何间隔的值，事实上，正负样本的几何间隔可以用下式统一表示：

$$ \gamma^{(i)} = y^{(i)} \left (\left( \frac {w}{\|w\|} \right)^T x^{(i)} + \frac {b} {\|w\|} \right) $$

如果$ \|\|w\|\| = 1 $，那么几何间隔将和函数间隔相等。对于几何间隔来说，如果将参数 (w, b) 增大为 (2w, 2b)，几何间隔的值是不会变的。

对训练集$ S = \{(x^{(i)}, y^{(i)}); i=1,...,m \} $，定义参数为 (w, b) 的几何间隔为各样本值中的几何间隔的最小值：

$$ \gamma = \min_{i=1,...,m} \gamma^{(i)} $$

### 最优间隔分类

根据对“间隔”的讨论，，我们可以用一个使间隔最大化的判定边界对正负样本进行分隔，而这将会是一个可信度最高的分隔。

假设一个训练集是线性可分的，即可以被一个超平面分隔正负样本。那么，我们怎么找到这个使几何间隔最大的超平面呢？可以用下面公式来描述这一问题：

$$ \begin{align*}
\max_{\gamma,w,b} & \gamma \\
s.t. &  y^{(i)}(w^T x^{(i)} + b) \ge \gamma, i=1,\ldots,m \\
&  \|w\|=1
\end{align*} $$

上述问题的解释是：求出最大的$ \gamma $，条件1是每个样本的函数间隔都大于等于 $ \gamma $，条件2是 $ \|\|w\|\|=1 $约束了函数间隔和几何间隔相等。求解出这个问题后，以$ \color{black}{w,b} $的值可以得到学习算法。

在上述问题中，约束 $\|\|w\|\|=1$并不是一个好的约束，并且上述问题不能用一个现成的优化软件去解决。因此将上述优化问题转化为一个更好的问题：

$$ \begin{align*}
\max_{\gamma,w,b} & \ \frac{\widehat{\gamma}}{\|w\|} \\
s.t. & \ y^{(i)}(w^T x^{(i)} + b) \ge \widehat{\gamma}, i=1,\ldots,m \\
\end{align*} $$

在这个问题中，我们最大化的目标是 $ \color{black}{\frac{\widehat{\gamma}}{\|\|w\|\|}} $，并且使每个训练样本的函数间隔不超过$\color{black}{\widehat{\gamma}}$，而最大化的目标即为几何间隔。这个问题丢掉了约束$\color{black}{\|\|w\|\|=1}$，但是这个目标$\color{black}{\frac{\widehat{\gamma}}{\|\|w\|\|}}$仍然是糟糕的，使这个问题仍然无法用优化软件去解决。

继续分析问题，由于参数$\color{black}{w,b}$可以以乘以任意的数而不会改变算法的结果，但是会影响函数间隔的结果。因此我们将函数间隔的最小值约束为1，那么几何间隔就为$\color{black}{\color{black}{\frac{\widehat{\gamma}}{\|\|w\|\|}} = 1/\|\|w\|\|}$，使几何间隔最大化也就是使$\color{black}{\|\|w\|\|}$最小化。所以有如下问题：

$$ \color{black}{\begin{align*}
\min_{\gamma,w,b} \& \ \frac{1}{2} \|w\|^2 \\
s.t. \& \ y^{(i)}(w^T x^{(i)} + b) \ge 1, i=1,\ldots,m \\
\end{align*}} $$

优化问题现在被转换为一个比较容易解决的问题，上述问题是一个凸二次目标，并且只有线性约束。这个问题的解就是最优间隔分类器。这个问题可以用现成的通用QP软件来解决。

### 拉格朗日对偶

现在我们讨论一些有关拉格朗日对偶的问题，它可以使我们得到上面的优化问题的对偶形式，这将是我们在最优间隔分类算法中引入**核**的关键，而**核**的引入将会使我们的算法在高维空间的更有效率。

考虑一个带约束的优化问题：

$$ \color{black}{\begin{align*}
\min_{w} & \ f(w) \\
s.t. & \ h_i(w)=0, i=1,\ldots,l. \\
\end{align*}} $$

现在我们引入拉格朗日乘子来解决这个问题，定义拉格朗日方程：

$$\color{black}{L(w,\beta)=f(w)+\sum\_{i=1}^l \beta\_i h\_i(w) }$$

这里，$\color{black}{\beta\_i}$就是拉格朗日乘子，现在令方程的偏导数为0：

$$ \color{black}{\frac{\partial L}{\partial w\_i} = 0; \frac{\partial L}{\partial \beta\_i} = 0} $$

然后解出$\color{black}{w} $和$ \color{black}{\beta} $

现在我们要在约束中增加一个不等式约束，再应用拉格朗日方式来解。这里只给出答案。

考虑下面的问题（这里被称为**原始优化问题**）：

$$ \color{black}{\begin{align*}
\min\_{w} \& \ f(w) \\
s.t. \& \ g\_i(w) \le 0, i=1,\ldots,k. \\
 \& \ h\_i(w)=0, i=1,\ldots,l. \\
\end{align*}} $$

现在定义通用拉格朗日方程：

$$ \color{black}{L(w, \alpha, \beta) = f(w) + \sum\_{i=1}^k \alpha\_i g\_i(w) + \sum\_{i=1}^l \beta\_i h\_i(w) } $$

考虑下面的值：

$$ \color{black}{\theta\_{\mathcal{P}} (w) = \max\_{\alpha,\beta:\alpha\_i \ge 0} L(w, \alpha,\beta)} $$

这里，$ \color{black}{\mathcal{P}} $下标表示的意思是**原始**。如果$\color{black}{w}$违反了任何原始约束，那么将会得到：

$$\color{black}{\theta\_{\mathcal{p}}(w) = \max\_{\alpha,\beta:\alpha\_i \ge 0} f(w) + \sum\_{i=1}^k \alpha\_i g\_i(w) + \sum\_{i=1}^l \beta\_i h\_i(w) = \infty} $$

因此，如果$\color{black}{w}$满足了所有的原始约束，则$\color{black}{\theta\_{\mathcal{P}} (w) = f(w)}$。那么，如果$\color{black}{w}$满足了原始约束，则$\color{black}{\theta\_{\mathcal{P}}}$会取得相同的值。如果不满足，则会得到一个正无穷的值。

$$ \color{black}{\min\_{w} \theta\_{\mathcal{P}} (w) = \min\_{w} \max\_{\alpha,\beta:\alpha\_i \ge 0} L(w, \alpha,\beta)} $$

可以看到上式和我们原始的问题有相同的解，定义$\color{black}{p^* = \min\_w \theta\_{\mathcal{P}} (w) }$，即为原始问题取得最优值的函数值。

现在，我们来看一个不同的问题，定义：

$$ \color{black}{\theta\_{\mathcal{D}} (\alpha, \beta) = \max\_{w} L(w, \alpha,\beta)} $$

这里，$ \color{black}{\mathcal{D}} $ 下标表示的意思是**对偶**。在原始问题中，我们的目标是改变$ \color{black}{\alpha, \beta} $求得最优值，在这个问题中，目标是改变$\color{black}{w}$来求最优值。

定义对偶问题：

$$\color{black}{\max\_{\alpha,\beta:\alpha\_i \ge 0} \theta\_\mathcal{D} (\alpha, \beta) = \max\_{\alpha,\beta:\alpha\_i \ge 0} \min\_w L(w, \alpha,\beta)}$$

这个公式和原始问题很相像，只是max和min调换了位置，我们定义对偶问题的最优值：$ \color{black}{d^* = \max\_{\alpha,\beta:\alpha\_i \ge 0}  \theta\_\mathcal{D} (w)} $。对偶问题和原始问题有什么联系呢？

$$ \color{black}{d^* = \max\_{\alpha,\beta:\alpha\_i \ge 0} \min\_w L(w, \alpha,\beta) \le \min\_{w} \max\_{\alpha,\beta:\alpha\_i \ge 0} L(w, \alpha,\beta) = p^*} $$

上式不做深入推理。然而，在特定的情况下会得到$\color{black}{d^* = p^*}$，这样，我们就可以通过对偶问题来求解原始问题了。现在分析什么情况下这两个值会相等。

先做一些假设，假设$\color{black}{f}$和$\color{black}{g\_i}$是凸函数(线性函数都属于凸函数)，假设$\color{black}{h\_i}$是仿射函数，再假设$\color{black}{g\_i}$是严格可执行的，即存在一些$\color{black}{w}$使$\color{black}{g\_i(w) < 0}$对所有的$\color{black}{i}$。

在做了这些假设之后，肯定存在$\color{black}{w^\*, \alpha^\*, \beta^\*}$ 使 $\color{black}{w^\*}$是原始问题的解，$\color{black}{\alpha^\*, \beta^\*}$是对偶问题的解。此时，$\color{black}{\alpha^\*, \beta^\*, w^\*}$满足KKT(Karush-Kuhn-Tucher)条件：

$$\color{black}{ \begin{eqnarray}
\label{num1} \frac{\partial}{\partial w\_i} L(w^\*, \alpha^\*, \beta^\*) \& = 0, i = 1,\ldots,n \\
\frac{\partial}{\partial \beta\_i} L(w^\*, \alpha^\*, \beta^\*) \& = 0, i = 1,\ldots,l \\
\label{num3} \alpha\_i^\* g\_i(w^\*) \& = 0, i = 1,\ldots,k \\
g\_i(w^\*) \& \le 0, i = 1,\ldots,k \\
\label{num5} \alpha^\* \& \ge 0, i = 1,\ldots,k \\
\end{eqnarray}}$$

如果$w^*, \alpha^*, \beta^*$满足KKT条件，那么它们就是原始问题和对偶问题的解。现在看公式$\eqref{num3}$，被称作KKT对偶互补条件，它意味着如果$\alpha\_i^* > 0$，那么$g\_i (w) = 0$。这个条件在后面将展示SVM只有一些支持向量点会起作用。

### 最优间隔分类求解

在最优间隔分类器的讨论中，列出了下面的优化问题来找到最优间隔分类器：

$$ \begin{align\*}
\min\_{\gamma,w,b} \& \ \frac{1}{2} \|\|w\|\|^2 \\
s.t. \& \ y^{(i)}(w^T x^{(i)} + b) \ge 1, i=1,\ldots,m \\
\end{align\*} $$

其中，约束条件可以改写为：

$$g\_i (w) = -y^{(i)} (w^T x^{(i)} + b) + 1 \le 0$$

注意在KKT条件中，讨论过如果$\alpha\_i > 0$只对训练集中那些函数间隔等于1的样本成立（对应着约束条件中的不等式取等号）。在下图中：实线表示的就是最大间隔的分隔超平面。


![最优间隔分类](/assets/images/machine-learing/ml4-3.jpg)

拥有最小间隔的点就是离判定边界最近的点；在上图中，有三个这样的点，在和判定边界平行的虚线上，也就是在优化问题中有三个$\alpha\_i$不等于0。这三个点被称为**支持向量**，并且支持向量的个数占整个训练集合的比例非常小。

这里可以构造出我们的优化问题的拉格朗日方程：

$$ \begin{equation}
\label{lagran} L(w,b,\alpha) = \frac{1}{2} \|\|w\|\|^2 - \sum\_{i=1}^m \alpha\_i [y^{(i)}(w^T(x)+b)-1]
\end{equation}$$

这里只有$\alpha_i$乘子，由于这个问题中只有不等式约束。

现在来找出这个问题的对偶形式。首先，需要最小化$L(w,b,\alpha)$在改变$w$和$b$的情况下(固定$\alpha$)，然后得到$\theta\_{\mathcal{D}}$
。现在需要将$L(w,b,\alpha)$对$w$和$b$的导数设为0。对$w$求导：

$$\nabla\_w L(w,b,\alpha) = w - \sum\_{i=1}^m \alpha\_i y^{(i)} x^{(i)} = 0$$

求解得到：

$$\begin{equation}
\label{lagran:dw} w = \sum\_{i=1}^m \alpha\_i y^{(i)} x^{(i)}
\end{equation}$$

将方程对$b$求导：

$$\begin{equation}
\label{lagran:db} \frac{\partial}{\partial b}L(w,b,\alpha) = \sum\_{i=1}^m \alpha\_i y^{(i)} = 0
\end{equation}$$

现在将公式$\eqref{lagran:dw}$中的$w$代入到拉格朗日方程(公式$\eqref{lagran}$)中，简化后得到：

$$L(w,b,\alpha) = \sum\_{i=1}^m \alpha\_i - \frac{1}{2} \sum\_{i,j=1}^m y^{(i)} y^{(j)} \alpha\_i \alpha\_j (x^{(i)})^T x^{(j)} - b \sum\_{i=1}^m \alpha\_i y^{(i)} $$

根据公式$\eqref{lagran:db}$，上式中最后一项为0，所以有：

$$L(w,b,\alpha) = \sum\_{i=1}^m \alpha\_i - \frac{1}{2} \sum\_{i,j=1}^m y^{(i)} y^{(j)} \alpha\_i \alpha\_j (x^{(i)})^T x^{(j)} $$

现在我们以$w, b$为参数对$L$求最小化的问题，将结果和$\alpha\_i \ge 0$以及公式$\eqref{lagran:db}$放到一起，可以得到对偶优化问题：

$$\begin{align\*}
\max\_\alpha \& W(\alpha) = \sum\_{i=1}^m \alpha\_i \frac{1}{2} \sum\_{i,j=1}^m y^{(i)} y^{(j)} \alpha\_i \alpha\_j (x^{(i)})^T x^{(j)} \\
s.t. \& \alpha\_i \ge 0, i=1,\ldots,m \\
 \& \sum\_{i=1}^m \alpha\_i y^{(i)} = 0 
\end{align\*}$$

现在可以验证$p^\* = d^\*$的条件和KKT条件（公式$\eqref{num1}$到公式$\eqref{num5}$）完全符合当前的优化问题。因此，我们可以求解对偶问题以求解原始问题。对偶问题是一个最大化问题，以$\alpha\_i$为参数。在后续课程将介绍如果求解这个问题，但是它确实是可解的。然后，根据公式$\eqref{lagran:dw}$可以得到$w^\*$，然后可以求出$b^\*$：

$$\begin{equation}
\label{solve:b} b^\* = -\frac{\max\_{i:y^{(i)}=-1} {w^\*\}^T x^{(i)} + \min\_{i:y^{(i)}=1} {w^\*\}^T x^{(i)}}{2}
\end{equation}$$

现在看公式$\eqref{lagran:dw}$，给出了$w$的最优值以$\alpha$作为参数。假设我们求出了模型的参数，现在需要对一个输入向量$x$作预测，那么需要计算$w^T x + b$，如果值大于0则预测结果为1. 但是代入公式$\eqref{lagran:dw}$：

$$\label{solve:predict} w^T x + b = \sum\_{i=1}^m \alpha\_i y^{(i)} \left \langle x^{(i)}, x \right \rangle + b$$

在上式中，预测过程需要计算输入向量和训练集中每一个向量的内积。而且，由于除了支持向量以外，其他的$\alpha\_i$都为0，所以，只需要计算输入向量$x$和所有的支持向量的内积即可。

在下一节课程中，将引入**核**的概念到分类问题中，将可以使算法在高维空间中更有效的工作。

### 参考

1. 斯坦福大学公开课--机器学习课程: <http://v.163.com/special/opencourse/machinelearning.html>
2. 课程课件下载地址: <http://cimg3.163.com/edu/open/ocw/jiqixuexikecheng.zip>
3. 函数间隔和几何间隔: <http://blog.csdn.net/csy463168656/article/details/8213975>
4. 凸优化: <http://zh.wikipedia.org/wiki/%E5%87%B8%E5%84%AA%E5%8C%96>
5. 对偶问题: <http://www.cs.cmu.edu/~epxing/Class/10701-06f/lectures/lecture9-annotated.pdf>
