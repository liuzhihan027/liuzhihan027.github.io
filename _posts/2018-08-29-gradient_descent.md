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

# 3. 梯度下降的思想

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

相应地,最大化函数的算法首先从一个随机初始点开始,计算梯度,在梯度方向(这是使函数增长最快的一个方向)上跨越一小步,再从一个新的初始点开始重复这个过程.同样,也可以在**相反**风向上逐步最小化函数.

![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-29-001.png)

如果一个函数有一个全局最小点,那么这个方法很可能会找到它.如果这个函数有多个(局部)最小点,那么这种方法可能找不到这个点,但可以通过多尝试一些初始点来重复运行这个方法.如果一个函数没有最小点,也许计算会陷入死循环.

# 4. 估算梯度

如果
$$ f $$
是单变量函数,那么它在
$$ x $$
点的导数衡量了当
$$ x $$
发生变化时,
$$ f(x) $$
变化了多少.导数通过差商的极限来定义:

```python
def difference_quotient(f, x, h):
    return (f(x + h) - f(x)) / h
```

其中
$$ h $$
趋近于0.

通过差商来求近似导数:

![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-30-001.png)

导数就是在点
$$ (x, f(x)) $$
的切线的斜率,而差商就是通过点
$$ (x, f(x)) $$
和点
$$ (x+h,f(x+h)) $$
的割线的斜率.当
$$ h $$
越来越小,割线与切线就越来越近(见上图).

很多函数可以精确地计算导数,比如平方函数square:

```python
def square( x ) :
    return x * x
```

它的导数为:

```python
def derivative( x ):
    return 2 * x
```

可以通过计算来确认,先显式地计算差商,再取极限.

如果算不出梯度或者不想算,Python中无法直接运算极限,但可以通过计算一个很小的变动
$$ e $$
的差商来估算微分.

```python
derivative_estimate = lambda x: difference_quotient(square, x, h=0.00001)

    # 精确值和估算值基本一致
    import matplotlib.pyplot as plt
    x = range(-10,10)
    plt.plot(x, map(derivative, x), 'rx')           # red  x
    plt.plot(x, map(derivative_estimate, x), 'b+')  # blue +
    plt.show()  
```

![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-30-002.png)

当
$$ f $$
是一个多变量函数时,它有多个偏导数,每个偏导数表示仅有一个输入变量发生微小变化时函数
$$ f $$
的变化.

我们把导数看成是其第
$$ i $$
个变量的函数,其他变量保持不变,以此来计算它第
$$ i $$
个偏导数:

```python
def partial_difference_quotient(f, v, i, h):
    # 仅对v的第i个元素增加h
    w = [v_j + (h if j == i else 0)
         for j, v_j in enumerate(v)]
         
    return (f(w) - f(v)) / h
```

再以同样的方式估算它的梯度函数:

```python
def estimate_gradient(f, v, h=0.00001):
    return [partial_difference_quotient(f, v, i, h)
            for i, _ in enumerate(v)] 
```

"差商估算法"的主要缺点是计算代价很大.如果
$$ v $$
长度为
$$ n $$
,那么estimate_gradient为了计算
$$ f $$
需要
$$ 2n $$
个不同的输入变量.如果你需要反复计算梯度,那需要你做很多额外的工作.


# 5. 使用梯度

很容易看出,当输入
$$ v $$
是零向量时,函数sum_of_squares取值最小.但如果不知道输入是什么,可以用梯度方法从所有的三维向量中找到最小值.先找出随机初始点,并在梯度的反方向以小步逐步前进,直到梯度变得非常非常小:

```python
def step(v, direction, step_size):
    # 在v的方向上移动步长
    return [v_i + step_size * direction_i
            for v_i, direction_i in zip(v, direction)]

# 梯度的平方
def sum_of_squares_gradient(v): 
    return [2 * v_i for v_i in v]

# 随机选择一个初始值
v = [random.randint(-10,10) for i in range(3)]

tolerance = 0.0000001

while True:
    #print v, sum_of_squares(v)
    gradient = sum_of_squares_gradient(v)   # 计算v的梯度
    next_v = step(v, gradient, -0.01)       # 取负的梯度步长
    if distance(next_v, v) < tolerance:     # 如果收敛了就停止
        break
    v = next_v                              # 如果没有汇合就继续

    print "minimum v", v
    print "minimum value", sum_of_squares(v)
```





















































