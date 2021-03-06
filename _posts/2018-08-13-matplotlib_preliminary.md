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

 ```python

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

# 5. 线图

```python

import matplotlib.pyplot as plt

# 处理中文
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

# 数据(方差,偏差,总误差)
variance     = [1,2,4,8,16,32,64,128,256]
bias_squared = [256,128,64,32,16,8,4,2,1]
total_error  = [x + y for x, y in zip(variance, bias_squared)]

xs = range(len(variance))

# 调用多次,可以在同一画布上添加多条线
plt.plot(xs, variance,     'g-',  label=u'方差')    # 绿色实线
plt.plot(xs, bias_squared, 'r-.', label=u'偏差的平方')      # 红色点虚线
plt.plot(xs, total_error,  'b:',  label=u'总误差') # 蓝色点线

# loc=9表示将图例放置在顶部中央
plt.legend(loc=9)
plt.xlabel(u"模型复杂度")
plt.title(u"偏差-方差权衡图")
plt.show()

```


  ![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-13-004.png)


# 6. 散点图


```python

import matplotlib.pyplot as plt

# 处理中文
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

# 数据(朋友数,在社交网站时间,用户标签)
friends = [ 70, 65, 72, 63, 71, 64, 60, 64, 67]
minutes = [175, 170, 205, 120, 220, 130, 105, 145, 190]
labels = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i']

plt.scatter(friends, minutes)

# 对每个点添加标记
for label, friend_count, minute_count in zip(labels, friends, minutes):
    plt.annotate(label,
                 xy=(friend_count, minute_count), # 将标记放在对应点上
                 xytext=(5, -5), # 需要部分偏离
                 textcoords='offset points')

plt.title(u"日分钟数与朋友数")

# 避免比例压缩
plt.axis("equal")
plt.xlabel(u"朋友的个数")
plt.ylabel(u"花在网站上的日分钟数")
plt.show()

```

这个函数
```python
plt.axis("equal")
```

这是避免比例压缩


  ![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-13-005.png)


如果直接使用画图,matplotlib会自动选择较合适的刻度,有时候可能会生成一个具有误导性的图


  ![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-13-006.png)
