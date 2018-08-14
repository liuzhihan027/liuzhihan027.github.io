---
layout: post
title:  "线性代数 入门"
categories: 数据科学入门
tags: 线性代数 初级
author: ZhL
---

* content
{:toc}

# 1. 简介


线性代数是数学的一个分支,研究向量空间,本文只介绍其中一些基础的部分,涵盖数据科学中大量的概念和技术,在今后的数据科学的学习中会有大量的应用.


# 2. 本文代码所引用的包及说明


引用的包:

```python
from __future__ import division  # 在使用"/"做除法时为精确除法
import re, math, random  # 正则,s数据函数以及随机数
import matplotlib.pyplot as plt  # 画图
from collections import defaultdict, Counter # 定义字典,计数器
from functools import partial # 偏函数
```


# 3. 向量

对于今后的数据分析而言,向量可以看作为有限维空间中的点,即使这样来做的话会令人感觉很抽象,但将数据数据表示为向量是目前来看非常好的方式.

最简单的方式就是将向量表示为数字的列表

## 3.1 两个向量相加
```python
# 向量相加
def vector_add(v, w):
    return [v_i + w_i for v_i, w_i in zip(v, w)]
```

## 3.2 两个向量相减
```python
# 两个向量相减
def vector_subtract(v, w):
    return [v_i - w_i for v_i, w_i in zip(v, w)]
```

## 3.3 多个向量相加

有时候我们会将多个向量进行相加操作,这时我们可以使用python中的高级的函数:
```python
# 多个向量相加
def vector_sum(vectors):
    return reduce(vector_add, vectors)
```

## 3.4 数乘向量

```python
# 数乘向量
def scalar_multiply(c, v):
    return [c * v_i for v_i in v]
```


## 3.5 多个向量均值

有时我们会需要计算一系列向量的均值(如聚类):

```python
# 多个向量的均值
def vector_mean(vectors):
    # 向量个数
    n = len(vectors)
    # 多个向量相加 * (1/向量个数)
    return scalar_multiply(1 / n, vector_sum(vectors))
```


## 3.6 向量点乘

向量的点乘为对应元素的分量乘积之和:

```python
# 向量的点乘
def dot(v, w):
    return sum(v_i * w_i for v_i, w_i in zip(v, w))
```

向量的点乘衡量了向量v在向量w的方向上的延伸程度,另一个解释是向量v在向量w上的投影所得到的向量长度.

## 3.6 向量平方和

通过点乘很容易计算出向量的平方和(同一个向量做点乘):

```python
# 向量的平方
def sum_of_squares(v):
    return dot(v, v)
```

## 3.6 向量的大小

向量的大小由向量的平方和开根号得到:

```python
# 向量的大小
def magnitude(v):
    return math.sqrt(sum_of_squares(v))
```


## 3.6 向量的距离

在有些时候我们需要计算两个向量间的距离,这里简单介绍下比较常用的欧几里得距离计算公式:

$$
d = \sqrt{(v_1-w_1)^2 + \cdots + (v_n-w_n)^2}
$$

 计算的方式是
 - 第一步,计算向量减法,生成新的向量
 - 第二步,计算新生成向量的大小

计算向量减法&&计算新向量的平方和:

```python
# 向量相减并求其平方和
def squared_distance(v, w):
    return sum_of_squares(vector_subtract(v, w))
```

 将平方和开方即得两向量间的欧氏距离:

 ```python
 # 将平方和开放,得两向量间的欧氏距离
 def distance(v, w):
     return math.sqrt(squared_distance(v, w))
 ```
