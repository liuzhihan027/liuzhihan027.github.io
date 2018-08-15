---
layout: post
title:  "概率 入门"
categories: 数据科学入门
tags: 概率论 初级
author: ZhL
---

* content
{:toc}

# 1. 简介

在本文中粗略地讲解下概率论,不对深入的细节做探讨.



# 2. 引用说明

```python
from __future__ import division
from collections import Counter
import math, random
import matplotlib.pyplot as plt
```

# 3. 不独立和独立

粗略的讲,如果E发生意味着F发生(或者F发生意味着E发生),我们就称事件E与事件F为不相互独立,否则,E与F就相互独立.

从数学角度讲,事件E和事件F独立意味着两个事件同时发生的概率等于它们分别发生的概率的乘积:

$$
P(E,F) = P(E)P(F)
$$

# 4. 条件概率

如果事件E与事件F独立,定义式如上方

如果两者不一定独立(并且F的概率不为0),那么E关于F的条件概率式如下:

$$
P(E|F) = P(E,F)/P(F)
$$

条件概率可以理解为,已知F发生,E会发生的概率.

 更常用的公式为上面公式的变形:

 $$
P(E,F) = P(E|F)P(F)
 $$

 如果E和F独立,则上式应该表示为:

 $$
P(E|F) = P(E)
 $$

 这个公式意味着,F是否发生并不会影响E是否发生的概率.

 举个例子,关于一个有两个孩子(性别未知)的家庭的例子.

 假设:
 - 每个孩子是男孩和是女孩的概率相同
 - 第二个孩子的性别概率与第一个孩子的性别概率相互独立

 那么,事件"没有女孩"的概率是1/4,事件"一个男孩,一个女孩"的概率为1/2,事件"两个女孩"的概率为1/4.

 现在新的问题,事件B"两个孩子都是女孩"关于事件G"大孩子是女孩"的条件概率是多少?用条件概率的定义式进行计算如下:

 $$
P(B|G) = P(B,G)/P(G) = P(B)/P(G)=1/2
 $$

 事件B与G的交集("两个孩子都是女孩并且大孩子是女孩")刚好是事件B本身.(一旦你知道两个孩子都是女孩,那大孩子必然是女孩.)

 继续问,事件"两个孩子都是女孩"关于事件"至少一个孩子是女孩"(L)的条件概率是多少?

 与前问相同的是,事件B和事件L的交集("两个孩子都是女孩,并且至少一个孩子是女孩")刚好是事件B,这意味着:

 $$
P(B|L) = P(B,L)/P(L) = P(B)/P(L)=1/3
 $$

 通过代码来以加以解释:

 ```python
 # 性别是随机的
 def random_kid():
    return random.choice(["boy", "girl"])

 # 初始化结果
 both_girls = 0
 older_girl = 0
 either_girl = 0

 # 随机进行10000次模拟实验
 random.seed(0)
 for _ in range(10000):
     younger = random_kid()
     older = random_kid()
     if older == "girl":
         older_girl += 1
     if older == "girl" and younger == "girl":
         both_girls += 1
     if older == "girl" or younger == "girl":
         either_girl += 1

 # 查看结果
 print "P(both | older):", both_girls / older_girl      # 0.514 ~ 1/2
 print "P(both | either): ", both_girls / either_girl   # 0.342 ~ 1/3
 ```

# 5. 贝叶斯定理

 对于数据工作者,贝叶斯定理是其最好的工具之一,它是条件概率的某种逆运算.假设我们需要计算事件E基于已发生的事件F的条件概率.两次使用条件概率的定义,可以得到下式:

 $$
P(E|F) = P(E,F)/P(F) = P(F|E)P(E)/P(F)
 $$

 事件F可以分割为两个互不重合的事件"F和E同时发生"与"F发生E不发生".我们使用
$$
\neg E
$$
表示非E(即E没有发生),有:


 $$
P(F) = P(F,E)+P(F,\neg E)
 $$

 因此:

 $$
P(E|F) = \frac{P(F|E)P(E)}{P(F|E)P(E)+P(F|\neg E)P(\neg E)}
 $$

 上式为贝叶斯定理常用的表达方式

# 6. 随机变量

来自百度百科上的解释,随机变量表示随机试验各种结果的实值单值函数。随机事件不论与数量是否直接有关，都可以数量化，即都能用数量化的方式表示.

随机事件数量化的好处是可以用数学分析的方法来研究随机现象。例如某一时间内公共汽车站等车乘客人数，电话交换台在一定时间内收到的呼叫次数，灯泡的寿命等等，都是随机变量的实例。


# 7. 连续分布

抛硬币对应的是**离散分布**,我们经常希望对连续结果的分布进行建模,例如**均匀分布**,函数对0到1之间的所有值都赋予相同的权重.

因为0和1之间有无数个数字,因而对每个点而言,赋予的权重几乎是零.因此我们用带**概率密度函数**的连续分布来表示概率,一个变量位于某个区间的概率等于概率密度函数在这个区间上的积分.

如果积分运算不直观,有一种更简单的理解方式:一个分布的密度函数为f,如果h很小,则变量的值落在x与x+h之间的概率接近h*f(x).

均匀分布的密度函数:

```python
# 均匀分布密度函数
def uniform_pdf(x):
    return 1 if x >= 0 and x < 1 else 0
```


我们还经常使用**累积分布函数**,这个函数给出了一个随机变量小于等于某一特定值的概率.下面生成均匀分布的累积函数:

```python
# 均匀分布累积函数
def uniform_cdf(x):
    # 返回一个均匀随机变量小于x的概率
    if x < 0:   return 0    # 均匀分布的随机变量不会小于0
    elif x < 1: return x    # e.g. P(X < 0.4) = 0.4
    else:       return 1    # 均匀分布的随机变量总是小于1
```


 ![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-15-001.png)


# 8. 正态分布

正态分布，也称“常态分布”，又名高斯分布,是一个在数学、物理及工程等领域都非常重要的概率分布，在统计学的许多方面有着重大的影响力。

正态分布的分布函数如下:

$$
f(x|\mu,\sigma) = \frac{1}{\sqrt{2\pi}\sigma}exp \left(  -\frac{(x-\mu)^2}{2\sigma^2}\right)
$$

可以这样实现:

```python
# 正态分布函数
def normal_pdf(x, mu=0, sigma=1):
    sqrt_two_pi = math.sqrt(2 * math.pi)
    return (math.exp(-(x-mu) ** 2 / 2 / sigma ** 2) / (sqrt_two_pi * sigma))
```

其概率密度函数:

```python
# 正态分布概率密度函数
def plot_normal_pdfs(plt):
    xs = [x / 10.0 for x in range(-50, 50)]
    plt.plot(xs,[normal_pdf(x,sigma=1) for x in xs],'-',label='mu=0,sigma=1')
    plt.plot(xs,[normal_pdf(x,sigma=2) for x in xs],'--',label='mu=0,sigma=2')
    plt.plot(xs,[normal_pdf(x,sigma=0.5) for x in xs],':',label='mu=0,sigma=0.5')
    plt.plot(xs,[normal_pdf(x,mu=-1)   for x in xs],'-.',label='mu=-1,sigma=1')
    plt.legend() # 图例位置
    plt.show()
```

 ![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-15-002.png)
