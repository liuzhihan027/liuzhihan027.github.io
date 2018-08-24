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

假设有一枚硬币,我们试图判断它是否均匀,即任何一面朝上的可能性是否相等.首先,假设硬币落地后正面朝上的概率为p,所以我们的零假设为硬币均匀,即p=0.5.我们要对比替代假设
$$ p \neq 0.5 $$
来检验这个假设.
