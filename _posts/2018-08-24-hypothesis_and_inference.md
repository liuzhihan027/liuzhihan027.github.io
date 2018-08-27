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

# 3. 抛硬币 案例

假设有一枚硬币,我们试图判断它是否均匀,即任何一面朝上的可能性是否相等.首先,假设硬币落地后正面朝上的概率为p,所以我们的零假设为硬币均匀,即
$$ p=0.5 $$
.我们要对比替代假设
$$ p \neq 0.5 $$
来检验这个假设.

具体来说,首先掷硬币n次,将出现正面朝上的次数记为
$$ X $$
.每次掷硬币都是一次伯努利试验,意味着
$$ X $$
是二项式随机变量
$$ Binomial(n,p) $$
,(如[概率论 入门](https://liuzhihan027.github.io/2018/08/15/probability_preliminary/ "概率论 入门")中所讲到的)可以使用正态分布来拟合:

```python
# 找到二项式参数为n和p的mu和sigma
def normal_approximation_to_binomial(n, p):
    mu = p * n
    sigma = math.sqrt(p * (1 - p) * n)
    return mu, sigma
```

只要一个随机变量服从正态分布,我们就可以用normal_cdf(正态分布的累积分布函数)来计算出一个实现数值位于(或不在)某一个特定区间的概率:

```python
# 正态cdf是一个变量在一个阈值一下的概率
normal_probability_below = normal_cdf

# 如果它不在阈值以下,就在阈值之上
def normal_probability_above(lo, mu=0, sigma=1):
    return 1 - normal_cdf(lo, mu, sigma)

# 如果它小于hi但不比lo小,那么它在区间之内
def normal_probability_between(lo, hi, mu=0, sigma=1):
    return normal_cdf(hi, mu, sigma) - normal_cdf(lo, mu, sigma)

# 如果不在区间之内,那么就在区间之外
def normal_probability_outside(lo, hi, mu=0, sigma=1):
    return 1 - normal_probability_between(lo, hi, mu, sigma)
```

或者反过来,找出非尾区域,或者找出均值两边的(对称)区域,这个区域恰好对应特定比例的可能性.比如,如果我们需要找出以均值为中心,覆盖60%可能性的区间,那我们需要找到两个截点,使上尾和下尾各覆盖20%的可能性(给中间留出60%):


```python
# 找到 P(Z<=z)= 概率 的 z 值
def normal_upper_bound(probability, mu=0, sigma=1):
    return inverse_normal_cdf(probability, mu, sigma)

# 找到 P(Z>=z)= 概率 的 z 值
def normal_lower_bound(probability, mu=0, sigma=1):
    return inverse_normal_cdf(1 - probability, mu, sigma)

# 返回关于均值对称的,包含指定概率的界限
def normal_two_sided_bounds(probability, mu=0, sigma=1):
    # 尾概率
    tail_probability = (1 - probability) / 2
    # 上界应该有尾概率在其上方
    upper_bound = normal_lower_bound(tail_probability, mu, sigma)
    # 下界应该有尾概率在其下方
    lower_bound = normal_upper_bound(tail_probability, mu, sigma)
    return lower_bound, upper_bound
```

具体来说,首先我们选择掷硬币
$$ n=1000 $$
次.如果关于均匀的原假设是正确的,那么
$$ X $$
近似服从正态分布,均值为50,标准差为15.8:

```python
# 二项分布的均值和期望
   mu_0, sigma_0 = normal_approximation_to_binomial(1000, 0.5)
   print "mu_0", mu_0 # 500.0
   print "sigma_0", sigma_0 # 15.8113883008
```

我们需要对**显著性**下定义--我们有多大的可能性犯**第一类错误**("容错").之这种情况下我们拒绝了原假设
$$ H_0 $$
,但实际上原假设是正确的.出于历史上的某些原因,可能性大小通常设定为5%或者1%.本文在此选择5%.

考虑这样的检验--如果
$$ X $$
落在以下区间以外,就拒绝原假设
$$ H_0 $$
:

```python

```
