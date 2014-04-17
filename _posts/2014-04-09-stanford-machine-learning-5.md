---
layout: post
title: "机器学习课程总结（五）"
description: "机器学习课程笔记第5篇，总结核的相关知识、L1软间隔分类器，SMO算法。"
category: machine-learning
tags: [机器学习, SVM, 核, Kernel, 软间隔分类器, SMO]
---

### 简介

机器学习课程笔记的第五篇。首先介绍了**核**的相关技术以及在SVM中的运用；然后简单介绍了**L1软间隔分类器**，使SVM算法能适应非线性可分的数据集；最后讲解了一个有效的SVM实现算法：**顺序最小优化算法**。

<!-- more -->

### 核

在线性回归中，我们将原始输入数据（如房屋的面积$x$）称为**输入属性**（input attributes），输入属性做的一些转换（如$x\^2, x\^3$）等 ，称为**输入特征**。我们用$\\phi$来表示从属性到特征的**特征映射**：

$$\\phi(x) = \\begin{bmatrix}
x \\\\
x\^2 \\\\
x\^3 
\\end{bmatrix}$$

在进行机器学习的训练时，有时使用映射后的特征$\\phi(x)$能取得更好的效果，此时，我们只要将公式中的$x$都替换成$\\phi(x)$。由于算法可以写成由内积组成的形式$\\left \\langle x, z \\right \\rangle$，意味着我们可以简单的用$\\left \\langle \\phi(x), \\phi(z) \\right \\rangle$来替换这个内积。这个内积就称为**核**：

$$K(x, z) = \\phi(x)\^T \\phi(x)$$

所以，算法出只要出现了$\\left \\langle x, z \\right \\rangle$，那么将它替换为$K(x,z)$之后，我们的算法就变成了用特征$\\phi(x)$来学习。

现在，只要给出$\\phi(x)$的实现，我们就可以计算$K(x,z)$，而且$K(x,z)$的计算速度是非常快的（虽然有时$\\phi(x)$的会很耗时）。在这种情况下，可以使SVM工作在一个高维空间中，并且不需要计算向量$\\phi(x)$。

看一个核的例子：

$$K(x,z) = (x\^T z)\^2$$

上面的公式可以转换为：

$$\\begin{align\*}
K(x,z) \& = \\left( \\sum\_{i=1}\^n x\_i z\_i \\right) \\left( \\sum\_{j=1}\^n x\_j z\_j \\right) \\\\
 \& = \\sum\_{i=1}\^n \\sum\_{j=1}\^n x\_i x\_j z\_i z\_j \\\\
 \& = \\sum\_{i,j=1}\^n (x\_i x\_j) (z\_i z\_j)
\\end{align\*}$$

可以看到，$K(x,z)=\\phi(x)\^T \\phi(z)$，特征映射为（假设属性为3维）：

$$\\phi(x) = \\begin{bmatrix}
x\_1 x\_1 \\\\
x\_1 x\_2 \\\\
x\_1 x\_3 \\\\
x\_2 x\_1 \\\\
x\_2 x\_2 \\\\
x\_2 x\_3 \\\\
x\_3 x\_1 \\\\
x\_3 x\_2 \\\\
x\_3 x\_3 \\\\
\\end{bmatrix}$$

注意计算高维的$\\phi(x)$需要$O(n\^2)$的时间复杂度，但是计算$K(x,z)$只需要耗费$O(n)$的时间复杂度。

看另一个核的例子：

$$\\begin{align\*}
K(x,z) \& = (x\^T z + c)\^2 \\\\
 \& = \\sum\_{i,j=1}\^n (x\_i x\_j) (z\_i z\_j) + \\sum\_{i=1}\^n (\\sqrt{2c} x\_i) (\\sqrt{2c} z\_i) + c\^2
\\end{align\*} $$

这个核对应着以下特征映射（假设属性为3维）：

$$\\phi(x) = \\begin{bmatrix}
x\_1 x\_1 \\\\
x\_1 x\_2 \\\\
x\_1 x\_3 \\\\
x\_2 x\_1 \\\\
x\_2 x\_2 \\\\
x\_2 x\_3 \\\\
x\_3 x\_1 \\\\
x\_3 x\_2 \\\\
x\_3 x\_3 \\\\
\\sqrt{2c} x\_1 \\\\
\\sqrt{2c} x\_2 \\\\
\\sqrt{2c} x\_3 \\\\
c
\\end{bmatrix}$$

上式中的参数$c$控制着$x\_i$（一阶）和$x\_ix\_j$（二阶）项的相关度。

对于更一般的核函数$K(x,z)=(x\^T z + c)\^d$对应着一个特征映射为$\\dbinom{n+d}{d}$维的特征空间，特征向量的每个元素都是$x\_i$的最高为d阶的组合。然而，尽管计算高维向量的内积需要很高的时间复杂度（$O(n\^d)$），但是计算$K(x,z)$只需要$O(n)$的时间复杂度，因此我们不需要将特征用高维的向量来表示。

现在从另一个角度来看核，直观上，如果$\\phi(x)$和$\\phi(z)$在空间中比较接近，那么内积$K(x,z)$的值就比较大，反之这个值会比较小。因此，可以认为$K(x,z)$是一个衡量$\\phi(x)$和$\\phi(z)$的相似度，或者说是$x$和$z$的相似度。

对于上述的理解，你可以想出一个合理的核来衡量$x$和$z$的相似度。如下面的公式：

$$K(x,z) = \\exp \\left( - \\frac {||x-z||\^2} {2 \\sigma\^2} \\right)$$

这是一个合理的值来衡量$x$和$z$的相似度，如果$x$和$z$距离较近，这个值会接近于1，相反会接近于0. 那么这个核能否被用于SVM算法中？对于这个公式是可以的。这个核被称为**高斯核**，对应着一个无限维的特征映射。然而，是否任意给出一个函数$K$，都是一个有效的核呢？即是否都存在一个$\\phi$，使$K(x,z) = \\phi(x)\^T \\phi(z)$。

假设所有的$K$都是一个有效的核并对应着的一特征映射函数$\\phi$，对于有限的m个特征点，构造矩阵$K\_{ij}= K(x\^{(i)}, x\^{(j)})$，这个矩阵称为**核矩阵**。那么如果$K$是一个有效的核，那么矩阵必须满足$K\_{ij}= K(x\^{(i)}, x\^{(j)})=\\phi(x\^{(i)})\^T \\phi(x\^{(j)}) = \\phi(x\^{(j)})\^T \\phi(x\^{(i)}) = K(x\^{(j)}, x\^{(i)}) = K\_{ji}$，因此矩阵$K$必须是对称的。

另外，让$\\phi\_k(x)$表示向量$\\phi(x)$的第k个坐标，我们发现对于任何向量$z$，都有：

$$\\begin{align\*}
z\^T K z \& = \\sum\_i \\sum\_j z\_i K\_{ij} z\_j \\\\
 \& = \\sum\_i \\sum\_j z\_i \\phi(x\^{(i)})\^T \\phi(x\^{(j)}) z\_j \\\\
 \& = \\sum\_i \\sum\_j z\_i \\sum\_k \\phi\_k(x\^{(i)})\^T \\phi\_k(x\^{(j)}) z\_j \\\\
 \& = \\sum\_k \\sum\_i \\sum\_j z\_i \\phi\_k(x\^{(i)})\^T \\phi\_k(x\^{(j)}) z\_j \\\\
 \& = \\sum\_k \\left( \\sum\_i z\_i \\phi\_k (x\^{(i)}) \\right) \^ 2 \\\\
 \& \\ge 0
\\end{align\*}$$

在上式中，由于$z$是任意的，所以$K$是半正定矩阵。

所以，如果$K$是一个合法的核，那么意味着对应的核矩阵是对称半正定矩阵。事实上，这个条件也是充分条件。

**Mercer定理** 给定一个$K: \\mathbb{R}\^n \\times \\mathbb{R}\^n \\mapsto \\mathbb{R}$，$K$是一个有效的核的充分必要条件是，对于任意的$\\{x\^{(1)},\\ldots,x\^{(m)}\\}, (m < \\infty)$，所对应的核矩阵都是对称半正定矩阵。

根据这个定理，我们可以检验一个函数$K$是不是一个有效的核，而不用去求解映射函数$\\phi$。

对于核来说，不仅仅存在于SVM算法中，对于任意的算法，只要计算中出现了输入属性内积的形式$\\left \\langle x, z \\right \\rangle$，都可以用核$K(x,z)$来代替，从而使算法可以在工作在高维空间中，和提高算法的性能。

### 软间隔分类器

到目前为止，关于SVM的推导都是建立在数据是线性可分的前提下的，或者数据在用映射函数$\\phi$映射到高维空间后是线性可分的。但是并不是所有的数据都是如此，由于有时候一些异常数据会导致无法找到一个超平面将数据分隔开，例如：下面左图中显示了一个最优间隔分类器，但是添加了一个异常数据之后（右图），使判定边界有一个旋转，而且导致分类器的间隔变小。

![异常数据分类器显示](/assets/images/machine-learing/ml5-1.jpg)

为了使SVM可以工作在非线性可分的数据集上，且保持对异常数据有较低的敏感度，我们修改优化问题如下（$l\_1$ 正规化）：

$$ \\begin{align\*}
\\min\_{\\gamma,w,b} \& \\ \\frac{1}{2} ||w||\^2 + C \\sum\_{i=1}\^m \\xi\_i \\\\
s.t. \& \\ y\^{(i)}(w\^T x\^{(i)} + b) \\ge 1 - \\xi\_i, i=1,\\ldots,m \\\\
 \& \\ \\xi\_i \\ge 0, i=1,\\ldots,m
\\end{align\*} $$

根据上面的公式，可以允许样本中出现函数间隔小于1的情况，如果一个样本的函数间隔为$1 - \\xi\_i$，我们会在优化目标中增加一些惩罚项$C\\xi\_i$。参数$C$控制着相关权重，使优化目标增大并保证大部分样本的函数间隔大于等于1.

如同前面所示，先写出拉格朗日方程：

$$L(w,b,\\xi,\\alpha,r) = \\frac{1}{2} w\^T w + C \\sum\_{i=1}\^m \\xi\_i - \\sum\_{i=1}\^m\\alpha\_i [y\^{(i)} (w\^T x + b) - 1 + \\xi\_i] - \\sum\_{i=1}\^m r\_i \\xi\_i$$

这里，$\\alpha\_i$和$r\_i$是拉格朗日乘子。我们不做推导，直接写出对偶问题：

$$ \\begin{align\*}
\\max\_{\\alpha} \& \\ W(\\alpha) = \\sum\_{i=1}\^m \\alpha\_i - \\frac{1}{2} \\sum\_{i,j=1}\^m y\^{(i)} y\^{(j)} \\alpha\_i \\alpha\_j \\left \\langle x\^{(i)}, x\^{(j)} \\right \\rangle \\\\
s.t. \& \\ 0 \\le \\alpha\_i \\le C, i=1,\\ldots,m \\\\
 \& \\ \\sum\_{i=1}\^m \\alpha\_i y\^{(i)} = 0, i=1,\\ldots,m
\\end{align\*} $$

和之前一样，在解出对偶问题之后，就可以对未知数据进行预测了。注意，对偶问题和之前的区别是约束由$0 \\le \\alpha\_i$变成了$0 \\le \\alpha\_i \\le C$，问题的解$b\^\*$也会不同。

另外，KKT对偶互补条件（这些条件将在下一节中判断SMO算法是否收敛）为：

$$ \\begin{align\*}
\\alpha\_i = 0 \& \\Rightarrow y\^{(i)} (w\^T x + b) \\ge 1 \\\\
\\alpha\_i = C \& \\Rightarrow y\^{(i)} (w\^T x + b) \\le 1 \\\\
0 < \\alpha\_i < C \& \\Rightarrow y\^{(i)} (w\^T x + b) = 1 \\\\
\\end{align\*} $$

现在，需要寻找一个算法来求解对偶问题，在下一节**顺序最小优化算法**中来完成。

### 顺序最小优化算法

顺序最小优化算法(SMO, sequential minimal optimization)，对SVM推导出的对偶问题给出了一个有效的解决方法。在介绍这个算法之前，先介绍另外一个算法：坐标上升算法(Coordinate ascent).

#### 坐标上升算法

考虑解决一个无约束的优化问题：

$$\\max\_\\alpha W(\\alpha\_1, \\alpha\_2, \\ldots, \\alpha\_m)$$

这里，$W$是一个和$\\alpha\_i$有关的函数，不考虑这个问题和SVM的关系。我们之前已经见过了两种优化算法：梯度上升和牛顿方法，现在介绍一种新的优化算法称为**坐标上升算法**：

$$\\begin{align\*}
\& \\	 Loop \\ until \\ convergence: \\{ \\\\
\& \\	\\	For \\ i=1,\\ldots,m, \\{ \\\\
\& \\	\\	\\	\\alpha\_i := \\arg \\max\_{\\hat{\\alpha}\_i} W (\\alpha\_1, \\ldots, \\alpha\_{i-1}, \\hat{\\alpha}\_i, \\alpha\_{i+1}, \\ldots, \\alpha\_m) \\\\
\& \\	\\	\\} \\\\
\& \\	\\}
\\end{align\*}$$

在上面的算法中，固定除了$\\alpha\_i$的其他参数，然后只改变$\\alpha\_i$来优化$W$。如果循环的$\\arg \\max$的求解过程比较快的话，那么这个算法还是比较高效的。下面的图表示了坐标上升算法的执行过程：

![坐标上升算法求解过程](/assets/images/machine-learing/ml5-2.jpg)

图中的椭圆是我们想要优化的二次函数的轮廓。坐标上升算法的初始值是$(2, -2)$，可以看出图中的优化路径找到了全局最优值。而且，由于算法是固定其他参数只改变一个参数，所以优化路径始终平等于坐标轴。

#### SMO

接下来我们简述一个SMO算法的推导过程，下面是我们想要求解的优化问题：

$$ \\begin{align}
\\max\_{\\alpha} \& \\ W(\\alpha) = \\sum\_{i=1}\^m \\alpha\_i - \\frac{1}{2} \\sum\_{i,j=1}\^m y\^{(i)} y\^{(j)} \\alpha\_i \\alpha\_j \\left \\langle x\^{(i)}, x\^{(j)} \\right \\rangle \\\\
\\label{ys1} s.t. \& \\ 0 \\le \\alpha\_i \\le C, i=1,\\ldots,m \\\\
\\label{ys2} \& \\ \\sum\_{i=1}\^m \\alpha\_i y\^{(i)} = 0, i=1,\\ldots,m
\\end{align} $$

在优化问题中，$\\alpha\_i$需要满足约束$\\eqref{ys1}, \\eqref{ys2}$。现在，我们来执行坐标上升算法的第一步，在固定$\\alpha\_2, \\ldots, \\alpha\_m$的情况下，改变$\\alpha\_1$的值，来优化目标。但是这样做可行吗？答案是不行，因为约束$\\eqref{ys2}$：

$$\\alpha\_1 y\^{(1)} = - \\sum\_{i=2}\^m \\alpha\_i y\^{(i)}$$

把上式两边都乘以$y\^{(1)}$，可以得到：

$$\\alpha\_1 = -y\^{(1)} \\sum\_{i=2}\^m \\alpha\_i y\^{(i)}$$

上式的推导使用了$(y\^{(1)})\^2 = 1$这一事实。因此，$\\alpha\_1$的值是由其他所有的$\\alpha\_i$的值来决定的，所以在固定$\\alpha\_2, \\ldots,\\alpha\_m$的情况下，$\\alpha\_1$的值是不能被改变的。

所以在SMO算法中，我们在每一步的过程中至少需要改变两个$\\alpha$的值，才能保定约束$\\eqref{ys2}$成立。算法描述：

Repeat till convergence {

  1. 选择下一步需要改变的参数对$\\alpha\_i$和$\\alpha\_j$(用一个启发式算法选择会使结果向全局最优值迈最大一步的一对参数)。
  2. 重新优化$W(\\alpha)$在改变参数$\\alpha\_i$和$\\alpha\_j$而其他参数固定的情况下。

}

SMO算法是一个高效的算法的原因是参数$\\alpha\_i$和$\\alpha\_j$可以被高效的更新，现在简述一下参数对的更新方法。

假设我们固定参数$\\alpha\_3, \\ldots, \\alpha\_m$，然后修改$\\alpha\_1$和$\\alpha\_2$的值来优化目标$W$，从约束$\\eqref{ys2}$，可以得到：

$$\\alpha\_1 y\^{(1)} + \\alpha\_2 y\^{(2)} = - \\sum\_{i=3}\^m \\alpha\_i y\^{(i)} $$

由于等式右边是固定的，我们可以用一个符号来表示这个固定值：

$$\\begin{equation}
\\label{eq1} \\alpha\_1 y\^{(1)} + \\alpha\_2 y\^{(2)} = \\zeta
\\end{equation}$$

现在可以将上式在坐标图中表示：

![SMO参数更新示意图](/assets/images/machine-learing/ml5-3.jpg)

由于约束$\\eqref{ys1}$，我们知道$\\alpha\_1, \\alpha\_2$必须位于图的$[0,C] \\times [0,C]$这个正方形内部。所以必须有$L \\le \\alpha\_2 \\le H$，否则会破坏约束。所以一般情况下都有一个最小值L（在这个例子中是0，但不代表所有情况）和一个最大值H，作为$\\alpha\_2$的取值范围，来确保满足所有约束条件。

根据方程$\\eqref{eq1}$，可以得到下式：

$$\\alpha\_1 = (\\zeta -\\alpha\_2 y\^{(2)} ) y\^{(1)} $$

上式也是用$(y\^{(1)})\^2 = 1$来推导的。这样，目标$W(\\alpha)$可以写成：

$$W(\\alpha\_1,\\alpha\_2, \\ldots,\\alpha\_m) = W \\left( (\\zeta -\\alpha\_2 y\^{(2)} ) y\^{(1)}，\\alpha\_2, \\ldots,\\alpha\_m \\right)$$

在上式中$\\alpha\_3, \\ldots,\\alpha\_m$可以被认为是常数，所以优化目标就变成了一个变量为$\\alpha\_2$的二次方程，意味着可以找合适的$a,b,c$使目标改写成$a \\alpha\_2\^2 + b \\alpha\_2 + c$，如果忽略掉"正方形"约束$\\eqref{ys1}$，我们就可以简单的将这个式子的导数设为0来求解最优值，假设结果为$\\alpha\_2\^{new,unclipped}$，那么和之前的约束条件一起：

$$\\alpha\_2\^{new} = \\begin{cases}
H, \& if \\alpha\_2\^{new,unclipped} > H \\\\
\\alpha\_2\^{new,unclipped}, \& if L \\le \\alpha\_2\^{new,unclipped} \\le H \\\\
L, \& if \\alpha\_2\^{new,unclipped} < L
\\end{cases}$$

这样就可以得到更新值$\\alpha\_2\^{new}$，进而求得更新值$\\alpha\_1\^{new}$。

关于SMO算法还有两点细节的技巧(关于这两点内在可以在参考文献4中说明)：

1. 怎么用启发式方法来选择下一步需要更新的$\\alpha\_i, \\alpha\_j$.
2. 怎样在SMO算法运行的时候更新b.

### 参考

1. 斯坦福大学公开课--机器学习课程: <http://v.163.com/special/opencourse/machinelearning.html>
2. 课程课件下载地址: <http://cimg3.163.com/edu/open/ocw/jiqixuexikecheng.zip>
3. 核: <http://en.wikipedia.org/wiki/Kernel_methods>
4. SMO算法: <http://research.microsoft.com/en-us/um/people/jplatt/smotr.pdf>
