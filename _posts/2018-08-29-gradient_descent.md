---
layout: post
title:  "梯度下降 入门"
categories: 数据科学入门
tags: 梯度下降 初级
author: ZhL
---

* content
{:toc}

# 1. 简介

梯度下降是迭代法的一种,可以用于求解最小二乘问题(线性和非线性都可以)。在求解机器学习算法的模型参数，即无约束优化问题时，梯度下降是最常采用的方法之一，另一种常用的方法是最小二乘法。在求解损失函数的最小值时，可以通过梯度下降法来一步步的迭代求解，得到最小化的损失函数和模型参数值。反过来，如果我们需要求解损失函数的最大值，这时就需要用梯度上升法来迭代了。在机器学习中，基于基本的梯度下降法发展了两种梯度下降方法，分别为随机梯度下降法和批量梯度下降法。






# 2. 引用说明

其中部分代码依赖
[线性代数 入门](https://liuzhihan027.github.io/2018/08/14/linear_algebra_preliminary/ "线性代数 入门")中所创建的函数

```python
from __future__ import division
from collections import Counter
# 依赖线性代数中的部分函数
from linear_algebra import distance, vector_subtract, scalar_multiply
import math, random
```

# 2. 梯度下降的思想

假设我们拥有某个函数 
$$ f $$
,这个函数输入一个实数向量,输出一个实数.一个简单的例子:

```python
def sum_of_squares(v):
    # 计算向量v中每个元素的平方和
    return sum(v_i ** 2 for v_i in v)
```

我们常常需要最大化(或最小化)这个函数.这意味着我们需要找出能计算出最大(或最小)可能值的输入
$$ v $$
.

对我们的函数来说,**梯度**(在微积分里,这表示偏导数向量)给出了输入值的方向,在这个方向上,函数增长得最快.

相应地,最大化函数的算法首先从一个随机初始点开始,计算梯度,在梯度方向()