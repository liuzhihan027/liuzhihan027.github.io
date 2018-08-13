---
layout: post
title:  "matplotlib 入门"
categories: 数据科学入门
tags:  数据可视化 matplotlib 初级
author: ZhL
---

* content
{:toc}

# 1. matplotlib简介

Matplotlib 可能是 Python 2D-绘图领域使用最广泛的套件。它能让使用者很轻松地将数据图形化，并且提供多样化的输出格式。这里将会探索 matplotlib 的常见用法。[官网](https://matplotlib.org/ "matplotlib官网")

# 2. 基本使用说明

```python
import matplotlib.pyplot as plt

# 数据(年份及其对应gdp值)
years = [1950, 1960, 1970, 1980, 1990, 2000, 2010]
gdp = [300.2, 543.3, 1075.9, 2862.5, 5979.6, 10289.7, 14958.3]

# 解决中文显示问题
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

# 创建线型图,横轴为年份,纵轴为gdp
plt.plot(years, gdp, color='green', marker='o', linestyle='solid')
plt.title(u"每年 GDP") # 添加标题

# 为y轴添加注释
plt.ylabel(u"十亿美元")
plt.show()

```
执行后显示结果:


 ![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-13-001.png)

# 3. 条形图

 ```python
 import matplotlib.pyplot as plt

 # 数据(电影名称和获奖次数)
 movies = ["Annie Hall", "Ben-Hur", "Casablanca", "Gandhi", "West Side Story"]
 num_oscars = [5, 11, 3, 8, 10]

 # 中文兼容
 plt.rcParams['font.sans-serif'] = ['SimHei']
 plt.rcParams['axes.unicode_minus'] = False

 # 横坐标
 xs = [i  for i, _ in enumerate(movies)]

 # 根据横纵坐标画图
 plt.bar(xs, num_oscars)
 plt.ylabel(u"获奖数量")
 plt.title(u"最喜爱的电影")

 # 电影名称标记x轴
 plt.xticks([i for i, _ in enumerate(movies)], movies)

 plt.show()

 ```

 显示结果:


  ![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-13-002.png)

# 4. 直方图


直方图的画法和条形图类似

```Python

import matplotlib.pyplot as plt
from collections import Counter


# 处理中文
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

# 数据(每个学生分数)
grades = [83,95,91,87,70,0,85,82,100,67,73,77,0]
# 划分分数等级
decile = lambda grade: grade // 10 * 10
# 每个分数级别对应的人数
histogram = Counter(decile(grade) for grade in grades)

# 第三个参数(8)设置条形宽度
plt.bar([x for x in histogram.keys()],
        histogram.values(),
        8)
# x轴范围(-5~105),y轴范围(0~5)
plt.axis([-5, 105, 0, 5])

# x轴注释(0, 10, ..., 100)
plt.xticks([10 * i for i in range(11)])
plt.xlabel(u"分数等级")
plt.ylabel(u"学生个数")
plt.title(u"考试分数分布图")
plt.show()

```


  ![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-13-003.png)
