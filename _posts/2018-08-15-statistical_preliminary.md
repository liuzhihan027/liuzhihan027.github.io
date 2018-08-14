---
layout: post
title:  "统计学 入门"
categories: 数据科学入门
tags: 统计学 初级
author: ZhL
---

* content
{:toc}

# 1. 简介

统计学是我们用来理解数据的数学和技术,这里进行简要的入门学习,将在以后的学习中不断丰富.




# 2. 引用说明

其中部分代码依赖
[线性代数 入门](https://liuzhihan027.github.io/2018/08/14/linear_algebra_preliminary/ "线性代数 入门")中所创建的函数

```python
from __future__ import division
from collections import Counter

# 依赖线性带式中的函数平方和和点乘
from linear_algebra import sum_of_squares, dot
import math
```

# 3. 描述单个数据集

对于较小的数据集来说最好的描述方式就是数据本身,同样的,也可以将数据转换成图标来展示,但对于较大的数据集来说,这样的方法并不适用,我们需要使用一些指标来描述数据.

较为简单的求数据集的极值,算法较为简单,在此不做深入学习.

## 3.1 中心倾向

### 3.1.1 均值

```python
# 均值
def mean(x):
    return sum(x) / len(x)
```

### 3.1.2 中位数

```python
# 计算中位数
def median(v):
    n = len(v) # 数据个数
    sorted_v = sorted(v) # 数据排序
    midpoint = n // 2 # 获取中间位置

    if n % 2 == 1:
        # 整除,返回中间位置对应的值
        return sorted_v[midpoint]
    else:
        # 未整除,获取其位置前后的值的均值
        lo = midpoint - 1
        hi = midpoint
        return (sorted_v[lo] + sorted_v[hi]) / 2
```

### 3.1.3 分位数

亦称分位点，是指将一个随机变量的概率分布范围分为几个等份的数值点，常用的有中位数（即二分位数）、四分位数、百分位数等.

```python
# 分位数
def quantile(x, p):
    # 获取指定分位的数组索引
    p_index = int(p * len(x))
    # 返回排序后的索引位置的值
    return sorted(x)[p_index]


# 二分位数(中位数)
quantile(arr, 0.50)
# 第一四分位数
quantile(arr, 0.25)
#第三四分位数
quantile(arr, 0.75)
```

### 3.1.4 众数
```python
# 众数
def mode(x):
    # 计数器计数
    counts = Counter(x)
    # 获取计数值最高的值
    max_count = max(counts.values())
    # 返回计数值最高的值的数组元素
    return [x_i for x_i, count in counts.iteritems()
            if count == max_count]
```
