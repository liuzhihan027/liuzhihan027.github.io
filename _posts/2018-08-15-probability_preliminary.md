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

 事件F可以分割为两个互不重合的事件"F和E同时发生"与"F发生E不发生".使用符号
$\neg E$
 指代"非E"(即"E没有发生"),有:

 $\neg E$

 $$
P(F) = P(F,E)+P(F,\neg E)
 $$

 因此:

 $$
P(E|F) = \frac{P(F|E)P(E)}{P(F|E)P(E)+P(F|\neg E)P(\neg E)}
 $$

 上式为贝叶斯定理常用的表达方式









<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
