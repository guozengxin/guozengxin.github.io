---
layout: post
title: "[hackerrank]Predicting Office Space Price"
description: "hackerrank题目, Polynomial Regression: Predicting Office Space Prices for Charlie"
category: machine-learning
tags: [machine-learning, polynomial regression]
comments: true
---

## 题目

题目地址：[Polynomial Regression: Predicting Office Space Prices for Charlie](https://www.hackerrank.com/challenges/predicting-office-space-price)

这是一个办公区域价格预测的问题，特征是办公楼的位置、公司的声誉、交通情况等，需要预测办公区的单位价格。在题目中指明了是多项式回归的算法：

> The prices per square foot, are (approximately) a polynomial function of the features in the observation table. This polynomial always has an order less than 4

## 题解

用python中的[sklearn][]包，利用[PolynomialFeatures][]模块，来实现多项式回归的算法。使用方法可能参考：[Polynomial interpolation](http://scikit-learn.org/stable/auto_examples/linear_model/plot_polynomial_interpolation.html)，在这个题目没到拿到10分，应该是多项式的维度上没有选择好，后面有时间可以再做尝试。

{% highlight python %}
#!/usr/bin/env python
# https://www.hackerrank.com/challenges/predicting-office-space-price

import sys
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures
from sklearn.pipeline import make_pipeline


def loadData():
    arr = sys.stdin.readline().strip().split(' ')
    F, N = [int(a) for a in arr]
    trainData = []
    testData = []
    for i in range(N):
        line = sys.stdin.readline()
        arr = line.strip().split(' ')
        trainData.append([float(a) for a in arr])
    T = input()
    for i in range(T):
        line = sys.stdin.readline()
        arr = line.strip().split(' ')
        testData.append([float(a) for a in arr])
    return F, N, trainData, T, testData


def main():
    F, N, trainData, T, testData = loadData()
    trainData = np.mat(trainData)
    testData = np.mat(testData)
    X = trainData[:, 0:F]
    y = trainData[:, F]
    minVar = 99999999
    finalDegree = 2
    for degree in [2, 3, 4]:
        model = make_pipeline(PolynomialFeatures(degree), LinearRegression())
        model.fit(X, y)
        predictR = model.predict(X)
        if minVar > np.var(predictR - y):
            finalDegree = degree
            minVar = np.var(predictR - y)
    model = make_pipeline(PolynomialFeatures(finalDegree), LinearRegression())
    model.fit(X, y)
    result = model.predict(testData)
    for r in result:
        print r[0]


if __name__ == '__main__':
    main()
{% endhighlight %}

[hackerrank]: https://www.hackerrank.com/
[sklearn]: http://scikit-learn.org/stable/
[PolynomialFeatures]:http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.PolynomialFeatures.html
