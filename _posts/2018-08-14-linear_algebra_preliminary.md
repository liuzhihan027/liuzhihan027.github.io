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

两个向量相加:
```python
# 向量相加
def vector_add(v, w):
    return [v_i + w_i for v_i, w_i in zip(v, w)]
```

两个向量相减:
```python
# 两个向量相减
def vector_subtract(v, w):
    return [v_i - w_i for v_i, w_i in zip(v, w)]
```

有时候我们会将多个向量进行相加操作,这时我们可以使用python中的高级的函数:
```python
# 多个向量相加
def vector_sum(vectors):
    return reduce(vector_add, vectors)
```

数乘向量:

```python
# 数乘向量
def scalar_multiply(c, v):
    return [c * v_i for v_i in v]
```
