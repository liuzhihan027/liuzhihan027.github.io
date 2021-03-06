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

如果运行以上程序,会发现,它总是止于一个非常接近[0,0,0]的
$$ v $$
值.tolerance值设定得越小,
$$ v $$
值就越接近[0,0,0].


# 6. 选择正确步长

尽管向梯度的方向移动的逻辑已经清除了,但移动多少还不明了.主流的选择方法有:

* 使用固定步长
* 时间增长逐步减小步长
* 在每一步中通过最小化目标函数的值来选择合适的步长

最后一种方法看上去不错,但它的计算代价也很大.可以尝试一系列步长,并选出使目标函数值最小的那个步长来求其近似值:

```python
step_sizes = [100, 10, 1, 0.1, 0.01, 0.001, 0.0001, 0.00001]
```

某些步长可能会导致函数的输入无效.所以,需要创建一个对无效输入值返回无限值(即这个值永远不会成为任何函数的最小值)的"安全应用"函数:

```python
def safe(f):
    # 定义一个新的函数来包装原函数(f)并返回它
    def safe_f(*args, **kwargs):
        try:
            return f(*args, **kwargs)
        except:
            return float('inf')         # 返回无穷(无限值)
    return safe_f
```

# 7. 综合

通常,有一些target_fn函数,需要对其进行最小化,也有梯度函数gradient_fn.比如,函数target_fn可能代表模型的残差,它是参数的函数.可能需要找到能使残差尽可能小的参数.

此外,假设(以某种方式)为参数theta_0设定了某个初始值,那么可以如下使用梯度下降法:

```python
def minimize_batch(target_fn, gradient_fn, theta_0, tolerance=0.000001):
    # 使用梯度下降寻找目标函数最小化的θ

    step_sizes = [100, 10, 1, 0.1, 0.01, 0.001, 0.0001, 0.00001]

    theta = theta_0                           # 设置初始值θ_0
    target_fn = safe(target_fn)               # 目标函数的安全函数
    value = target_fn(theta)                  # 目前正在最小化的值

    while True:
        gradient = gradient_fn(theta)  
        next_thetas = [step(theta, gradient, -step_size)
                       for step_size in step_sizes]

        # 选择一个最小化误差函数的函数  
        next_theta = min(next_thetas, key=target_fn)
        next_value = target_fn(next_theta)

        # 如果小于阈值则认为是收敛,返回
        if abs(value - next_value) < tolerance:
            return theta
        else:
            theta, value = next_theta, next_value
```

批量梯度下降,在每一步梯度计算中,会搜索整个数据集(因为target_fn代表整个数据集的残差).

如果我们需要**最大化**某个函数,这时只需要最小化这个函数的负值(其对应的梯度函数也要取负值):

```python
# 函数取负值
def negate(f):
    return lambda *args, **kwargs: -f(*args, **kwargs)

# 函数导数取负值
def negate_all(f):
    return lambda *args, **kwargs: [-y for y in f(*args, **kwargs)]

# 批量梯度上升(梯度下降取负值)
def maximize_batch(target_fn, gradient_fn, theta_0, tolerance=0.000001):
    return minimize_batch(negate(target_fn),
                          negate_all(gradient_fn),
                          theta_0,
                          tolerance)
```

# 8. 随机梯度下降法

上方使用的方法,通过最小化某种形式的残差来选择模型参数.如果使用批量梯度下降法,每个梯度计算都需要预测并计算整个数据集的梯度,这使得每一步都会耗费很长时间.

这些残差函数常常具有**可加性**,意味着整个数据集上的预测残差恰好是每个数据点的预测残差之和.

这种情形下,使用**随机梯度下降法**,它每次仅计算一个点的梯度,并向前跨一步.这个计算会反复循环,直到达到一个停止点.

在每个循环中,都会在整个数据集上按照一个随机序列迭代:

```python
def in_random_order(data):
    indexes = [i for i, _ in enumerate(data)]  # 建立索引列表
    random.shuffle(indexes)                    # 随机打乱数据
    for i in indexes:                          # 返回列表中的数据
        yield data[i]

```

对每个数据点都会进行一步梯度计算.这种方法含有这样一种可能性,也许会在最小值附近一直循环下去,所以,每当停止获得改进,都会减小步长并最终退出:

```python
# 随机梯度下降
def minimize_stochastic(target_fn, gradient_fn, x, y, theta_0, alpha_0=0.01):
    data = zip(x, y)
    theta = theta_0                             # 初始值猜测
    alpha = alpha_0                             # 初始步长
    min_theta, min_value = None, float("inf")   # 当前最小值
    iterations_with_no_improvement = 0

    # 如果循环超过100次仍无改进,则停止
    while iterations_with_no_improvement < 100:
        value = sum( target_fn(x_i, y_i, theta) for x_i, y_i in data )

        if value < min_value:
            # 如果找到新的最小值,记住它,并返回最初的步长
            min_theta, min_value = theta, value
            iterations_with_no_improvement = 0
            alpha = alpha_0
        else:
            # 逐步缩小步长,否则没有改进
            iterations_with_no_improvement += 1
            alpha *= 0.9

        # 在每个数据点上向梯度方向前进一步
        for x_i, y_i in in_random_order(data):
            gradient_i = gradient_fn(x_i, y_i, theta)
            theta = vector_subtract(theta, scalar_multiply(alpha, gradient_i))

    return min_theta
```

随机梯度下降通常比批量梯度下降要快,如果需要获得最大化的结果:

```python
def maximize_stochastic(target_fn, gradient_fn, x, y, theta_0, alpha_0=0.01):
    return minimize_stochastic(negate(target_fn),
                               negate_all(gradient_fn),
                               x, y, theta_0, alpha_0)
```
