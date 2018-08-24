---
layout: post
title:  "假设与推断 入门"
categories: 数据科学入门
tags: 统计学 初级 假设推断
author: ZhL
---

* content
{:toc}

# 1. 简介

假设检验是数理统计学中根据一定假设条件由样本推断总体的一种方法。具体作法是：根据问题的需要对所研究的总体作某种假设，记作H0；选取合适的统计量，这个统计量的选取要使得在假设H0成立时，其分布为已知；由实测的样本，计算出统计量的值，并根据预先给定的显著性水平进行检验，作出拒绝或接受假设H0的判断。常用的假设检验方法有u—检验法、t检验法、χ2检验法(卡方检验)、F—检验法，秩和检验等。






# 2. 引用说明

其中部分代码依赖
[概率论 入门](https://liuzhihan027.github.io/2018/08/15/probability_preliminary/ "概率论 入门")中所创建的函数

```python
from __future__ import division
# 依赖概率论中的正态分布相关函数
from probability import normal_cdf, inverse_normal_cdf
import math, random
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


## 3.2 离散度

离散程度,是指通过随机地观测变量各个取值之间的差异程度,用来衡量风险大小的指标.

### 3.2.1 极差

度量离散度最简单的指标就是极差:

```python
# 极差
def data_range(x):
    return max(x) - min(x)
```


### 3.2.2 方差

方差是在概率论和统计方差衡量随机变量或一组数据时离散程度的度量。概率论中方差用来度量随机变量和其数学期望（即均值）之间的偏离程度。统计中的方差（样本方差）是每个样本值与全体样本值的平均数之差的平方值的平均数。在许多实际问题中，研究方差即偏离程度有着重要意义。

使用M代表均值,n代表数据个数,方差的计算公式为:

$$
s^2 = \frac{(x_1-M)^2 + (x_2-M)^2 + \cdots + (x_n-M)^2}{n}
$$

这里我们计算方差的方法为:

- 计算均值
- 将原始向量每个值减去均值,形成新的向量
- 计算新向量的平方和
- 使用平方和除以向量元素个数

计算均值+将原始向量每个值减去均值,形成新的向量:

```python
# (向量每个元素-向量元素均值)形成新的向量
def de_mean(x):
    # 向量元素均值
    x_bar = mean(x)
    return [x_i - x_bar for x_i in x]
```

计算新向量的平方和+使用平方和除以向量元素个数

```python
# 方差
def variance(x):
    # 向量元素个数
    n = len(x)
    # 计算均值+将原始向量每个值减去均值,形成新的向量
    deviations = de_mean(x)
    # 计算新向量的平方和+使用平方和除以向量元素个数
    return sum_of_squares(deviations) / (n - 1)
```

由代码可以看出我们除以的是(n-1)而不是n,这是为了使样本方差的估计为无偏估计,具体原因及推倒公式将在以后给出.


### 3.2.3 标准差

标准差,又常称均方差，是离均差平方的算术平均数的平方根，用σ表示。标准差是方差的算术平方根。标准差能反映一个数据集的离散程度。平均数相同的两组数据，标准差未必相同。

标准差的计算方法:

```python
# 标准差
def standard_deviation(x):
    # 方差的平方根
    return math.sqrt(variance(x))
```

### 3.2.4 四分位差

是上四分位数（Q3，即位于75%）与下四分位数（Q1，即位于25%）的差, 相对来说这种计算不容易受到小部分异常值的干扰.

四分位差计算方法:

```python
# 四分位差
def interquartile_range(x):
    #上四分位数-下四分位数
    return quantile(x, 0.75) - quantile(x, 0.25)
```


# 4. 描述相关

## 4.1 协方差

协方差在概率论和统计学中用于衡量两个变量的总体误差。而方差是协方差的一种特殊情况，即当两个变量是相同的情况。

协方差表示的是两个变量的总体的误差，这与只表示一个变量误差的方差不同。 如果两个变量的变化趋势一致，也就是说如果其中一个大于自身的期望值，另外一个也大于自身的期望值，那么两个变量之间的协方差就是正值。 如果两个变量的变化趋势相反，即其中一个大于自身的期望值，另外一个却小于自身的期望值，那么两个变量之间的协方差就是负值。

 协方差的计算公式:

 $$
Cov(X,Y) = \frac{1}{n}\sum_{i=1}^n(x_i - E(X))(y_i - E(Y))
 $$

 其中E(X),E(Y)我们使用X和Y的均值

计算过程:

- 对每个向量计算,均值,和其每个值减去均值所形成的新的向量
- 将上述两个新生成的矩阵做点乘
- 将点乘的结果除以元素个数

协方差计算:

```python
# 协方差
def covariance(x, y):
    # 元素个数
    n = len(x)
    # 点乘(均值,和其每个值减去均值所形成的新的向量)/(n-1)
    return dot(de_mean(x), de_mean(y)) / (n - 1)
```

这里也是使用的(n - 1)而不是使用n,同计算方差的原因一致.


## 4.2 相关系数

相关系数是最早由统计学家卡尔·皮尔逊设计的统计指标，是研究变量之间线性相关程度的量，一般用字母 r 表示。由于研究对象的不同，相关系数有多种定义方式，较为常用的是皮尔逊相关系数。

 其计算公式为:

 $$
r(X,Y) = \frac{Cov(X,Y)}{\sqrt{Var[ X ]Var[ Y ]}}
 $$

计算方式为协方差除以两个变量的标准差:

```python
# 相关系数
def correlation(x, y):
    # 计算标准差
    stdev_x = standard_deviation(x)
    stdev_y = standard_deviation(y)
    # 需要满足两个向量的标准差均大于0
    if stdev_x > 0 and stdev_y > 0:
        #协方差/标准差x/标准差y
        return covariance(x, y) / stdev_x / stdev_y
    else:
        return 0 # 如果含标准差为0则协方差为0
```






































<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
