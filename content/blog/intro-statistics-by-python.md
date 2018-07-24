+++
date = "2018-02-07T14:15:32+08:00"
title = "数据分析预备"
showonlyimage = false
image = "/img/blog/intro-statistics-by-python/math.png"
draft = true
weight = 51
+++

复习统计基础知识、常用编程技巧
<!--more-->

统计分为 descriptive 和 inference： 描述统计学是对已有数据的压缩总结，推论统计学是试图超越现有数据，对更大范围进行假设推断。统计学中最为重要的是培养数学思维，它能避免人类对过大/小数据的感知无能，在上下文中理解和提取出关键问题，从而作出正确决策。

和[集中趋势](https://en.wikipedia.org/wiki/Central_tendency)相关的测量：左右对称，且最常见数值分布在最中间的数据集称作正态分布，与此相关的计算是平均值，而挑选中间值则可以去除少数异类对整体数据的影响，中值和平均值重合时数据的[偏度](https://en.wikipedia.org/wiki/Skewness)为 0 。最后[众数](https://en.wikipedia.org/wiki/Mode_(statistics))是从数据集中找出最最常出现的一个或多个样本(甚至不是数)。

和[离散程度](https://en.wikipedia.org/wiki/Statistical_dispersion)常用“距离”度量：比如[全距](https://en.wikipedia.org/wiki/Range_(statistics))描述的是全体数据、[四分位距](https://en.wikipedia.org/wiki/Interquartile_range)(IQR)则是去除了极限值后,仅考虑中间 50% 核心数据范围。上述测量仅仅考虑数据中的两点，而[方差](https://en.wikipedia.org/wiki/Variance)“综合考虑”了所有的数据点，即计算每个实际值和平均值之差为边长的正方形面积后，求和，再除以样本数量，记作 Var(X) 或 σ²(读作 sigma squared) 注意最终结果的计量单位是被平方的结果。为了把测量单位降维，将方差开方得到[标准差](https://en.wikipedia.org/wiki/Standard_deviation)。总之，这些值都是越大表明数据的离散程度越大。

> 为了获得 Unbiased Variance，最后会除以 n-1

对无法区分顺序/互做加减云散的分类数据(Category Data)，可用(相对)频度分布对每个类别进行统计、在[柱状图](https://en.wikipedia.org/wiki/Bar_chart)上可视化后进行描述比较。而最常见的定量数据(Quantitavie Data)，其间距相等，可以划分多个连续范围(Binning)后用直方图(Histogram)统计描述。为了不丧失任何原始数据，还可以用 [Stem-and-Leaf图](https://en.wikipedia.org/wiki/Stem-and-leaf_display) 来标记所有的数据点。[箱线图](https://en.wikipedia.org/wiki/Box_plot) 综合了中值，最大最小值，四分位距，异常点等信息。

## f-Strings

% formatting .format()  f''

x = 10
print('The answer is %d today' % x)
print('The answer is {0} today'.format(x))
print('The answer is {x} today'.foramt(x=x))
print(f'The answer is {x} today')
print(f'The answer is {x: 08d} today')
print(f'The answer is {x ** 2: 08d} today')

raise ValueError(f'Expected {x!r} to a float not a {type(x).__name__}')


## Counter vs dict

from collections import Counter

c = Counter()
c['dragons'] += 1

c = Counter('red green red blue red blue green'.split())
c.most_common(1)
c.most_common(2)
list(c.elements()) # same group together
list(c)
list(c.values())
list(c.items())


## stdev vs pstdev in statistics module

from statistics import mean, median, mode, stdev, pstdev

mean([50, 52, 53])
median([50, 52, 53])
mode([50, 51, 51, 51, 52, 53])


## list concatenation vs numpy plus

s = [10, 20, 30]
t = [40, 50, 60]
u = s + t
u[:2] +  u[-2:]

s = 'abracadabra'
i = s.index('c')
s[i] == s[4]
s.count('a') == 5

s = [10, 5, 70, 2]
s.sort()
t = sorted(s)

sorted('cat')

## lambda

100 + (lambda x: x**2)(5) + 50 == 175

f = lambda x, y: 3*x + y
f(3, 8) == 17

x = 5
y = 2
f = lambda: x ** y
f() # make a promise in future

## Chained comparisons
x = 15
x > 6 and x < 20
6 < x < 20



## random and distribution

from random import *

seed(someint)
random()
random()
random()
seed(sameint)
random()
random()
random()


data = [triangular(1000, 1100) for i in range(1000)]
mean(data)
stdev(data)

data = [uniform(1000, 1100) for i in range(1000)]
mean(data)
stdev(data)

data = [gauss(100, 15) for i in range(1000)]
mean(data)
stdev(data)

data = [expovariate(20) for i in range(1000)]
mean(data)
stdev(data)


## discret distribution

from random import choice, choices, sample, shuffle

outcomes = ['win', 'lose', 'draw', 'play again', 'double win']

choice(outcomes)
choice(outcomes)
choice(outcomes)

choices(outcomes, k=10)

from collections import Counter

Counter(choices(outcomes, k=10))
Counter(choices(outcomes, k=10000))
Counter(choices(outcomes, [5, 4, 3, 2, 1], k=10000))

outcomes
shuffle(outcomes)
outcomes

sample(outcomes, k=4) # choices (dup) vs sample (no dup)
sample(outcomes, k=4)
sample(outcomes, k=4)

sorted(sample(range(1, 57), k=6)) # lattory

sample(outcomes, k=1)[0]
choice(outcomes)

shuffle(outcomes); outcomes
sample(outcomes, k=len(outcomes))

参考文档


> - YouTube playlist [Crash Course on Statistics](https://www.youtube.com/watch?v=zouPoc49xbk&list=PL8dPuuaLjXtNM_Y-bUAhblSAdWRnmBUcr)
> - Raymond Hettinger (2017) [Modern Python](https://github.com/rhettinger/modernpython)
> - Dominic Walliman (2017-02-01) [The Map of Mathematics](https://youtu.be/OmJ-4B-mS-Y)

封面图片来自 [Shiba Inu - Math](https://dribbble.com/shots/3818487-Shiba-Inu-Math) <a href="https://dribbble.com/vaneltia"><i class="fa fa-dribbble" aria-hidden="true"></i> Alfrey Davilla | vaneltia</a>
