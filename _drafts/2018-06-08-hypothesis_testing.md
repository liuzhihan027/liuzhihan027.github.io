---
layout: post
title:  "假设检验"
categories: mathematical_statistics
tags:  mathematical_statistics
author: ZhL
---

* content
{:toc}

# 1. 假设检验简介

假设检验(Hypothesis Testing)是数理统计学中根据一定假设条件由样本推断总体的一种方法。

具体作法是：

>- 根据问题的需要对所研究的总体作某种假设，记作H0；

>- 选取合适的统计量，这个统计量的选取要使得在假设H0成立时，其分布为已知；

>- 由实测的样本，计算出统计量的值，并根据预先给定的显著性水平进行检验，作出拒绝或接受假设H0的判断。

常用的假设检验方法有z检验法、t检验法、χ2检验法(卡方检验)、F—检验法，秩和检验等。

# 2. z检验

Z检验（Z Test）是一般用于大样本（即样本容量大于30）平均值差异性检验的方法。它是用标准正态分布的理论来推断差异发生的概率，从而比较两个平均数的差异是否显著。在国内也被称作u检验。

当已知标准差时，验证一组数的均值是否与某一期望值相等时，用Z检验。

$$ I^2 = \int_{0}^{+\infty}e^{-x^2}dx \cdot \int_{0}^{+\infty}e^{-x^2}dx \\
 = \int_{0}^{+\infty}e^{-x^2}dx \cdot \int_{0}^{+\infty}e^{-y^2}dy\; \\
 = \int_{0}^{+\infty}dx \cdot \int_{0}^{+\infty}e^{-(x^2+y^2)}dy\; \\
 = \underset{0\leq x\leq+\infty;0\leq x\leq+\infty } \iint\quad e^{-(x^2+y^2)}dxdy
 \xrightarrow{使用极坐标变换}
 \int_{0}^{\frac{\pi}{2}}d\theta \int_{0}^{+\infty} e^{-r^2} \cdot rdr \\
 =\dfrac{\pi}{2} \cdot \int_{0}^{+\infty} e^{-r^2}(-\dfrac{1}{2})(-2rdr) \\
 =\dfrac{\pi}{2} \cdot (-\dfrac{1}{2})\int_{0}^{+\infty} e^{-r^2}d(-r^2) \\
 =-\dfrac{\pi}{4}(e^{-r^2} | _0^{+\infty})\\
 =\dfrac{\pi}{4}\\
 故:I=\dfrac{\sqrt{\pi}}{4}
   $$



# 3. t检验


# 4. 卡方检验


# 5. χ2检验


# 6. f检验


# 7. 秩和检验
