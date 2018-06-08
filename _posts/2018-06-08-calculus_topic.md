---
layout: post
title:  "微积分题目"
categories: calculus
tags:  calculus question
author: ZhL
---

* content
{:toc}

# 1. 一场用微积分结果求解过程

$$ I^2 = \int_{0}^{+\infty}e^{-x^2}dx \cdot \int_{0}^{+\infty}e^{-x^2}dx
 = \int_{0}^{+\infty}e^{-x^2}dx \cdot \int_{0}^{+\infty}e^{-y^2}dy\; \\
 = \int_{0}^{+\infty}dx \cdot \int_{0}^{+\infty}e^{-(x^2+y^2)}dy\; \\
 = \underset{0\leq x\leq+\infty;0\leq y\leq+\infty } \iint\quad e^{-(x^2+y^2)}dxdy
 \xrightarrow{使用极坐标变换}
 \int_{0}^{\frac{\pi}{2}}d\theta \int_{0}^{+\infty} e^{-r^2} \cdot rdr \\
 =\dfrac{\pi}{2} \cdot \int_{0}^{+\infty} e^{-r^2}(-\dfrac{1}{2})(-2rdr)
 =\dfrac{\pi}{2} \cdot (-\dfrac{1}{2})\int_{0}^{+\infty} e^{-r^2}d(-r^2) \\
 =-\dfrac{\pi}{4}(e^{-r^2} | _0^{+\infty})
 =\dfrac{\pi}{4}\\
 I=\dfrac{\sqrt{\pi}}{4}
$$


<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
