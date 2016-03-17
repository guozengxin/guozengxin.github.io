---
layout: post
title: "[hackerrank]Predicting House Prices"
description: "hackerrank题目, Multivariate Linear Regression: Predicting House Prices for Charlie."
category: machine-learning
tags: [machine-learning, linear regression]
comments: true
---

## 题目

题目地址：[Multivariate Linear Regression: Predicting House Prices for Charlie](https://www.hackerrank.com/challenges/predicting-house-prices)

这个题一个多特征的线性回归问题，问题也是机器学习中最典型的房价预测的问题。

## 题解

[hackerrank][]允许使用python中的[sklearn][]包，我在解这个题目时用了这个包中的线性回归的算法。可以参考<http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html>

下面是我的代码：

```python
#!/usr/bin/env python
# https://www.hackerrank.com/challenges/predicting-house-prices

import sys
import numpy as np
from sklearn import linear_model


def main():
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
    trainData = np.mat(trainData)
    lr = linear_model.LinearRegression()
    lr.fit(trainData[:, 0:F], trainData[:, F])
    for r in lr.predict(np.mat(testData)):
        print r[0]

if __name__ == '__main__':
    main()
```


[hackerrank]: https://www.hackerrank.com/
[sklearn]: http://scikit-learn.org/stable/
