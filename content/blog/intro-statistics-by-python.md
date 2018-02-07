+++
date = "2018-02-07T14:15:32+08:00"
title = "数据分析基础"
showonlyimage = false
image = "/img/blog/intro-statistics-by-python/math.png"
draft = false
weight = 51
+++

格式化字符串、统计模块
<!--more-->

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


> - Raymond Hettinger (2017) [Modern Python](https://github.com/rhettinger/modernpython)

封面图片来自 [Shiba Inu - Math](https://dribbble.com/shots/3818487-Shiba-Inu-Math) <a href="https://dribbble.com/vaneltia"><i class="fa fa-dribbble" aria-hidden="true"></i> Alfrey Davilla | vaneltia</a>
