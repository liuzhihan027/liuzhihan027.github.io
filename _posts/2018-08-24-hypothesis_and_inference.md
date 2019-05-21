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

假设有一枚硬币,我们试图判断它是否均匀,即任何一面朝上的可能性是否相等.首先,假设硬币落地后正面朝上的概率为p,所以我们的零假设为硬币均匀,即
$$ p=0.5 $$
.我们要对比替代假设
$$ p \neq 0.5 $$
来检验这个假设.

具体来说,首先掷硬币n次,将出现正面朝上的次数记为
$$ X $$
.每次掷硬币都是一次伯努利试验,意味着
$$ X $$
是二项式随机变量
$$ Binomial(n,p) $$
,(如[概率论 入门](https://liuzhihan027.github.io/2018/08/15/probability_preliminary/ "概率论 入门")中所讲到的)可以使用正态分布来拟合:

```python
# 找到二项式参数为n和p的mu和sigma
def normal_approximation_to_binomial(n, p):
    mu = p * n
    sigma = math.sqrt(p * (1 - p) * n)
    return mu, sigma
```

只要一个随机变量服从正态分布,我们就可以用normal_cdf(正态分布的累积分布函数)来计算出一个实现数值位于(或不在)某一个特定区间的概率:

```python
# 正态cdf是一个变量在一个阈值一下的概率
normal_probability_below = normal_cdf

# 如果它不在阈值以下,就在阈值之上
def normal_probability_above(lo, mu=0, sigma=1):
    return 1 - normal_cdf(lo, mu, sigma)

# 如果它小于hi但不比lo小,那么它在区间之内
def normal_probability_between(lo, hi, mu=0, sigma=1):
    return normal_cdf(hi, mu, sigma) - normal_cdf(lo, mu, sigma)

# 如果不在区间之内,那么就在区间之外
def normal_probability_outside(lo, hi, mu=0, sigma=1):
    return 1 - normal_probability_between(lo, hi, mu, sigma)
```

或者反过来,找出非尾区域,或者找出均值两边的(对称)区域,这个区域恰好对应特定比例的可能性.比如,如果我们需要找出以均值为中心,覆盖60%可能性的区间,那我们需要找到两个截点,使上尾和下尾各覆盖20%的可能性(给中间留出60%):


```python
# 找到 P(Z<=z)= 概率 的 z 值
def normal_upper_bound(probability, mu=0, sigma=1):
    return inverse_normal_cdf(probability, mu, sigma)

# 找到 P(Z>=z)= 概率 的 z 值
def normal_lower_bound(probability, mu=0, sigma=1):
    return inverse_normal_cdf(1 - probability, mu, sigma)

# 返回关于均值对称的,包含指定概率的界限
def normal_two_sided_bounds(probability, mu=0, sigma=1):
    # 尾概率
    tail_probability = (1 - probability) / 2
    # 上界应该有尾概率在其上方
    upper_bound = normal_lower_bound(tail_probability, mu, sigma)
    # 下界应该有尾概率在其下方
    lower_bound = normal_upper_bound(tail_probability, mu, sigma)
    return lower_bound, upper_bound
```

具体来说,首先我们选择掷硬币
$$ n=1000 $$
次.如果关于均匀的原假设是正确的,那么
$$ X $$
近似服从正态分布,均值为50,标准差为15.8:

```python
# 二项分布的均值和期望
   mu_0, sigma_0 = normal_approximation_to_binomial(1000, 0.5)
   print "mu_0", mu_0 # 500.0
   print "sigma_0", sigma_0 # 15.8113883008
```

我们需要对**显著性**下定义--我们有多大的可能性犯**第一类错误**("容错").之这种情况下我们拒绝了原假设
$$ H_0 $$
,但实际上原假设是正确的.出于历史上的某些原因,可能性大小通常设定为5%或者1%.本文在此选择5%.

考虑这样的检验--如果
$$ X $$
落在以下区间以外,就拒绝原假设
$$ H_0 $$
:

```python
print "normal_two_sided_bounds(0.95, mu_0, sigma_0)", normal_two_sided_bounds(0.95, mu_0, sigma_0)
# (469.01026640487555, 530.9897335951244)
```

假设
$$ p $$
实际上等于0.5(即,此时
$$ H_0 $$
成立),那么我们有5%的可能性观测到
$$ X $$
落在区间之外,这正是我们想要的显著性.换句话说,如果
$$ H_0 $$
为真,那么20次检验中大约有19次会得出正确的结果.

我们常常对检验的**势**有兴趣,它指的是不犯**第2类错误**的概率.第2类错误指原假设
$$ H_0 $$
是错的,但我们的检验结果没有拒绝原假设(即"纳伪").为了衡量统计的势,我们需要精确衡量
$$ H_0 $$
是错的**意味着什么**.(仅仅知道
$$ p $$
不是0.55不足以为
$$ X $$
的分布提供足够的信息.)具体来说,假如
$$ p $$
实际上是0.55,那么掷硬币的结果会稍微多偏向正面朝上.

在这种情形下,我们这样计算检验的势:

```python
# 基于假设p是0.5时95%的边界
lo, hi = normal_two_sided_bounds(0.95, mu_0, sigma_0)
print "lo", lo # 469.010266405
print "hi", hi # 530.989733595

# 基于 p = 0.55的真实mu和sigma
mu_1, sigma_1 = normal_approximation_to_binomial(1000, 0.55)
print "mu_1", mu_1 # 550.0
print "sigma_1", sigma_1 # 15.7321327226

# 第二类错误意味着我们没有拒绝原假设
# 这会在X仍然在最初的区间时发生
type_2_probability = normal_probability_between(lo, hi, mu_1, sigma_1)
power = 1 - type_2_probability
print "type 2 probability", type_2_probability # 0.113451998705
print "power", power # 0.886548001295
```

如果我们把原假设变为掷硬币的结果不会偏重于正面朝上,即
$$ P \leq 0.5 $$
,在这种情况下,我们使用**单边检验**.如果
$$ X $$
远大于50,我们就拒绝原假设,如果
$$ X $$
小于50,我们就不拒绝原假设.因此,显著性为5%的检验需要使用normal_probability_below来找出小于95%的概率对应的截点:

```python
hi = normal_upper_bound(0.95, mu_0, sigma_0)
print "hi", hi # 526.007358524 (< 531,因此我们在上尾需要更多的概率)
type_2_probability = normal_probability_below(hi, mu_1, sigma_1)
power = 1 - type_2_probability
print "type 2 probability", type_2_probability # 0.0636205196693
print "power", power # 0.936379480331
```

这是更有效的检验.如果
$$ X $$
小于469,我们就不再拒绝
$$ H_0 $$
(如果
$$ H_1 $$
为真,这不太可能发生),当
$$ X $$
在526和531之间时则拒绝
$$ H_0 $$
(如果
$$ H_1 $$
为真,这很有可能发生).

进行上述检验的另一种方式涉及**p值**.我们不再基于某个概率截点选择临界值,而是计算概率--假设
$$ H_0 $$
正确--我们可以找到一个至少与我们实际观测到的值一样极端的值.

对于硬币是否均匀的双面检验,我们可以做如下计算:

```python
def two_sided_p_value(x, mu=0, sigma=1):
    if x >= mu:
        # 如果x大于均值,tail表示比x大多少
        return 2 * normal_probability_above(x, mu, sigma)
    else:
        # 如果x比均值小,tail表示比x小多少
        return 2 * normal_probability_below(x, mu, sigma)
```

如果我们希望看到结果中有530次为正面朝上,可以这样计算:

```python
print "two_sided_p_value(529.5, mu_0, sigma_0)", two_sided_p_value(529.5, mu_0, sigma_0)  # 0.062077215796
```

为什么使用529.5而不用530?这是所谓的**连续矫正**.它反映了一个事实,即对从掷硬币结果中看到530次正面朝上的概率而言,normal_probability_between(529.5,530.5,mu_0,sigma_0)是比normal_probability_between(530,531,mu_0,sigma_0)更好的估计.

相应的,normal_probability_above(529.5,nu_0,sigma_0)是看到至少530次正面朝上概率的更好估计,如图:

![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-15-004.png)

回到主要探讨的问题上来,验证这种观点是否合理的一个方法是模拟:

```python
def count_extreme_values():
    # 对极端值进行计数
    extreme_value_count = 0
    for _ in range(100000):
        num_heads = sum(1 if random.random() < 0.5 else 0    # 正面朝上计数
                        for _ in range(1000))                # 一千次投掷
        if num_heads >= 530 or num_heads <= 470:             # 计算达到极值的频率
            extreme_value_count += 1

    return extreme_value_count / 100000
# 0.06305
```

因为
$$ p $$
值大于5%的显著性,所以我们不能拒绝原假设.如果我们看到了532次正面朝上,那么相应的
$$ p $$
值为:

```python
two_sided_p_value(531.5, mu_0, sigma_0) # 0.0463452878378
```

它小于5%的显著性,因此我们拒绝原假设.它正好是和之前相同的检验,只是计算统计量的方法稍有不同.

同样我们有:

```python
upper_p_value = normal_probability_above
lower_p_value = normal_probability_below
```

对于单边检验,如果我们看到525次正面朝上,那么可以计算:

```python
upper_p_value(525, mu_0, sigma_0) # 0.0606288577258
```

这意味着我们会拒绝原假设.如果我们看到527次正面朝上,相应计算如下:

```python
upper_p_value(526.5, mu_0, sigma_0) # 0.0468683950886
```

根据结果,我们会拒绝原假设.

再调用函数normal_probability_above计算
$$ p $$
值之前,需要确定你的数据大致上服从正态分布.如果数据本身不是正态分布的,那结果就毫无意义.

对正态分布的检验方法有好几种,绘图是不错的首选方案.



# 4. 置信区间

我们一直在对正面朝上的概率
$$ p $$
进行假设检验,这是未知的"正面朝上"分布的**参数**.对假设检验,我们还有第三种方法:在参数的观测值附近建立**置信区间**.

例如,我们可以通过计算每次抛掷对应的伯努利随机变量的均值来估计不均匀硬币的概率--正面朝上记为1,背面朝上记为0,取这一系列伯努利随机变量的平均值.如果我们观测的1000次抛掷中有525次正面朝上,那么我们可以估计出
$$ p $$
等于0.525.

但是这个估计的**可信度**有多大呢?如果我们已知
$$ p $$
的精确值,那么根据中心极限定理(见[概率论 入门](https://liuzhihan027.github.io/2018/08/15/probability_preliminary/ "概率论 入门")),伯努利随机变量的均值近似服从正态分布,其中均值为
$$ p $$
,标准为:

```python
math.sqrt(p * (1 - p) / 1000)
```

这里,
$$ p $$
是未知的,所以我们使用估值:

```python
p_hat = 525 / 1000
mu = p_hat
sigma = math.sqrt(p_hat * (1 - p_hat) / 1000) # 0.0158
```

这种计算是不严格的,但运用广泛.借助正态近似我们得出结论:以下区间包含真实参数
$$ p $$
的可能性为95%:

```python
normal_two_sided_bounds(0.95, mu, sigma) # [0.4940, 0.5560]
```

这是关于区间的解释,不是关于
$$ p $$
值的解释.需要这样理解:如果重复实验很多次,其中95%的"真"参数(每次都相同)会落在观测到的置信区间(每次可能会都不同)中.

注意,我们没有得出不均匀硬币的结论,因为0.5落入了我们的置信区间.

如果我们观察到的是540次正面朝上,那么相应计算为:

```python
p_hat = 540 / 1000
mu = p_hat
sigma = math.sqrt(p_hat * (1 - p_hat) / 1000) # 0.0158
normal_two_sided_bounds(0.95, mu, sigma) # [0.5091,0.579]
```

在这种情况下,"均匀硬币"没有落入置信区间.("均匀硬币"的假设没有通过检验.如果假设是真的,那需要在95%的时间中都能通过.)


# 5. P-hacking


如果一个程序仅有5%的时间错误地拒绝了原假设,那么根据定义,5%的时间会错误地拒绝原假设:

```python
def run_experiment():
    # 掷硬币100次,真 = 头概率,假 = 尾概率
    return [random.random() < 0.5 for _ in range(1000)]

def reject_fairness(experiment):
    # 使用5%作为显著水平
    num_heads = len([flip for flip in experiment if flip])
    return num_heads < 469 or num_heads > 531

random.seed(0)
experiments = [run_experiment() for _ in range(1000)]
num_rejections = len([experiment
                      for experiment in experiments
                      if reject_fairness(experiment)])

print num_rejections, "rejections out of 1000" # 46
```

这意味着如果你有意找出"显著"结果,那么总是可以的.只要对数据的假设检验次数足够多,就总有某些会表现出显著性.移除右边的那些异常值,就可以得到小于0.05的
$$ p $$
值.(这里与[统计学 入门](https://liuzhihan027.github.io/2018/08/15/statistical_preliminary// "统计学 入门")中的相关性有些类似.)

这就是所谓的P-hacking,它某种程度上是"基于
$$ p $$
值框架的推断
"的结果.

如果从事数据方面的工作,就要在审查数据之前确定你的假设,就要在做假设之前整理好数据,并且要牢记,
$$ p $$
值并不是靠直觉得出的.


# 6. 案例:运行A/B测试


一个广告厂商开发了一种新的能量饮料,广告部门希望在广告A("口味好")和广告B("营养均衡")之间进行选择.

这样我们需要做**实验**,对网站访问者随机放送不同的广告,并记录每个广告的点击数.

如果1000个看到广告A的人中有990个人点击广告,而1000个看到广告B的人中只有10个点击,你可以确认A是更棒的广告.但倘若区别并不如此分明,可以使用统计推断进行选择.

假设
$$ N_A $$
个人看到广告A,其中
$$ n_A $$
个人点击广告.每次广告浏览都是一次伯努利试验,其中
$$ p_A $$
是点击广告A的概率.然后(如果
$$ N_A $$
足够大.此处就足够大)我们知道
$$ n_A/N_A $$
是近似服从正态分布的随机变量,其中均值为
$$ p_A $$
,标准差为
$$
\sigma_A = \sqrt{p_A(1-p_A)/N_A}
$$
.同样,
$$ n_B/N_B $$
是近似服从正态分布的随机变量,均值为
$$ p_B $$
,标准差为
$$
\sigma_B = \sqrt{p_B(1-p_B)/N_B}
$$
:

```python
def estimated_parameters(N, n):
    p = n / N
    sigma = math.sqrt(p * (1 - p) / N)
    return p, sigma
```

如果我们假设这两个正态分布互相独立(这个假设是合理的,因为每个伯努利试验也是独立的),那么它们的差也是正态分布的,其中均值为
$$ p_B - p_A $$
,标准差为
$$ \sqrt{\sigma_A^2 + \sigma_B^2} $$


这某种程度上有些欺骗性.只有在标准差已知的条件下数学推理才正确.我们从数据中估计参数,这意味着我们实际中应该使用
$$ t $$
分布.但如果数据集足够大,正态分布和
$$ t $$
分布之间的差别可以忽略不计.

这意味着我们可以检验
$$ p_A $$
和
$$ p_B $$
相等(即
$$ p_A - p_B $$
等于零)这个**原假设**,具体方式如下:

```python
def a_b_test_statistic(N_A, n_A, N_B, n_B):
    p_A, sigma_A = estimated_parameters(N_A, n_A)
    p_B, sigma_B = estimated_parameters(N_B, n_B)
    return (p_B - p_A) / math.sqrt(sigma_A ** 2 + sigma_B ** 2)
```

这应该近似一个标准正态分布.

比如,如果"口味好"的广告从1000次浏览中获得200次点击量,则统计量等于:

```python
z = a_b_test_statistic(1000, 200, 1000, 180) # -1.1403464899
```

如果两个均值实际上相等,那么看到如此大的差异的概率为:

```python
two_sided_p_value(z) # 0.254141976542
```

这算出的数值很大,以至于你不可以得出有差距的结论.另一方面,如果"营养均衡"仅仅获得150次点击量,则:

```python
z = a_b_test_statistic(1000, 200, 1000, 150) # -2.9488391231
two_sided_p_value(z) # 0.00318969970622
```

# 7. 贝叶斯推断

我们所看到的处理方式都包含对检验所做的与概率有关的陈述:"如果原假设正确,那么你观测到的极端统计量的概率仅有3%."

推断的一个替代方法是将未知参数视为随机变量.从参数的**先验分布**出发,再利用观测数据和贝叶斯定理计算出更新后的**后验分布**.不再对检验本身给出概率推断,二是对参数本身给出概率判断.

比如,如果未知参数是概率(就想掷硬币的例子),我们使用**Beta分布**作为先验分布,Beta分布仅对0和1赋值:

```python
def B(alpha, beta):
    # 归一化操作,使总概率为1
    return math.gamma(alpha) * math.gamma(beta) / math.gamma(alpha + beta)

def beta_pdf(x, alpha, beta):
    if x < 0 or x > 1:          # [0, 1]以外没有权重
        return 0
    return x ** (alpha - 1) * (1 - x) ** (beta - 1) / B(alpha, beta)
```

一般来说,以上分布的权重中心为:

```python
alpha / (alpha + beta)
```

alpha和beta越大,分布就越"紧密".

例如,如果alpha和beta都是1,那么刚好是均匀分布(以0.5为中心,非常分散).如果alpha比beta大很多,那么大多数权重接近1.如果alpha比beta小很多,那么大多数权重接近零.(几种不同的Beta分布)

![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-28-001.png)

先假设一个先验分布p.如果对硬币是否均匀不预设立场,那么将alpha与beta都设定为1.或者如果我们坚信硬币有55%的可能正面朝上,就选择让alpha等于55,beta等于45.

然后我们多次掷起硬币,结果有
$$ h $$
次正面朝上,有
$$ t $$
次背面朝上.根据贝叶斯定理,
$$ p $$
的先验分布仍然是Beta分布,但参数分别为
$$ alpha + h $$
和
$$ beta + t $$
.

后验分布也是Beta分布,这并非偶然.二项分布给出了正面朝上的数字,Beta是二项分布的共轭先验分布.这意味着,无论你何时使用从相关的二项分布中得到的观测值更新Beta先验分布,还是会得到一个Beta后验分布.

假设掷硬币10次并且观测到3次正面朝上.

如果从均匀分布的先验开始(有时候不会采取硬币均匀的立场),那么你的后验分布为Beta(4,8),中心为0.33.如果你认为所有的可能性都相等,那么最好的猜测就会非常接近观测到的概率.

如果从Beta(20,20)开始(表名硬币大致上是均匀的),那么后验分布为Beta(23,27),中心为0.46,这表明可能硬币稍稍倾向于背面朝上.

如果从Beta(30,10)开始(表明硬币是不均匀的,即有75%的可能会正面朝上),那么后验分布为Beta(33,17),中心为0.66.这种情况下,仍然相信正面朝上的概率会大一些,只是没有一开始那么坚定了.这几个不同的后验分布:

![image](https://github.com/liuzhihan027/liuzhihan027.github.io/raw/master/images-folder/2018-08-28-002.png)

如果多次掷硬币,无论最初选择了什么样的先验分布,先验分布对后验分布的影响会越来越小,直到最后得到(几乎)相同的后验分布.

比如,无论最初对掷硬币的结果有怎样的倾向猜想,当看到2000次掷硬币的结果中有1000次正面朝上时,都会很难维持原先的看法(除非极端选择了Beta(1000000,1)这样的先验分布).

有趣的是,这允许对假设"基于先验分布和已观测数据,正面朝上的概率介于49%~51%的可能性仅有5%"做出概率判断.
