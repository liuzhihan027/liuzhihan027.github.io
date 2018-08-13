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
