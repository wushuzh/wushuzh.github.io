+++
date = "2017-11-06T21:18:21+08:00"
title = "流利 python"
showonlyimage = false
image = "/img/blog/play-pythonic-cards/playing-cards.png"
draft = false
weight = 103
+++

愿敲代码终有一天能比斗地主更 666
<!--more-->

Luciano Ramalho著有 fluent python 一书。最近看了他在 [PyBay 2017 的分享](https://youtu.be/M4gPxbo6G6k) 关于 namedtuple 的介绍，核心内容都在 [Jupyter Notebook](https://github.com/fluentpython/think-like-a-pythonista/blob/master/think-like-a-pythonista.ipynb) 上。

首先是在自己的机器上安装 jupyter ，并设置 Chrome 为默认打开浏览器:

{{< highlight console >}}
$ sudo pacman -S jupyter-notebook jupyter-nbconvert
$ jupyter notebook --generate-config
Writing default config to: ~/.jupyter/jupyter_notebook_config.py

# manuall edit ~/.jupyter/jupyter_notebook_config.py 
# and uncomment item 
#    c.NotebookApp.browser = '/usr/bin/google-chrome-stable %s'

$ jupyter notebook
{{< /highlight >}}

为了让编辑更为方便，建议打印 [cheatsheet](https://www.cheatography.com/weidadeyue/cheat-sheets/jupyter-notebook/) 贴在案前。

我试图尝试将 jupyter notebook 键入博客，试了网上的各种方法，感觉最简单的是建立一个 gist 利用 github 自动转换到 html 服务嵌入文中，但要看着顺眼还需要 CSS 调整，或许结合 nbconvert 定制化模板是个比较好的解决方案，下文还是通过普通 python 代码展示概念。

### 轻量级 namedtuple 类
 
通过 collections.namedtuple 可用来定义集合中的一个元素类型，他们不像标准类那么“重”，但却能增加不少类对象的功能，定义使用起来都非常方便。

假设定义每张扑克牌，
- 第一个参数，类名 Card
- 第二个参数，定义对象属性/字段: 这里是牌值和花色，放在一个字符串中并用空格/逗号分割即可，如'rank suit'

{{< highlight pycon >}}

>>> Card = namedtuple('Card', 'rank suit')
>>> beer_card = Card('7', 'diamonds')
>>> beer_card
Card(rank='7', suit='diamonds')
>>> rank, suit = beer_card
>>> rank, suit
('7', 'diamonds')
>>> beer_card.suit
'diamonds'
>>> beer_card.rank
'7'

{{< /highlight >}}

### 成套花色

上面定义好了每张牌，接下来就可以定义一幅牌的集合了。尤其是 list comprehensive 快速生成一个 list ，并且两个 lists 还可以做相加: 合并为一个 list。

{{< highlight pycon >}}
>>> SUITS = 'clubs diamonds spades hearts'.split()
>>> SUITS
['clubs', 'diamonds', 'spades', 'hearts']
>>> RANKS = [ str(n) for n in range(2, 11)] + list('JQKA')
>>> RANKS
['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A']
>>> CARDS = [Card(r, s) for s in SUITS for r in RANKS]
>>> len(CARDS)
52
>>> CARDS[:3]
[Card(rank='2', suit='clubs'), Card(rank='3', suit='clubs'), Card(rank='4', suit='clubs')]

{{< /highlight >}}

### 集合类 Deck

只要有一些特定的 Dunder 方法，我们就能在应用中对这个类的对象使用类似数组通过索引或区间取值的特有语法:

> 注意我们在初始化的时候，对 CARDS 做了一次拷贝，这样一个不可变更的序列才能安全的做各种操作。

{{< highlight pycon >}}
>>> class Deck:
...   def __init__(self):
...     self._cards = list(CARDS)
...
>>> class Deck:
...   def __init__(self):
...     self._cards = list(CARDS)
...   def __len__(self):
...     return len(self._cards)
...   def __getitem__(self, i):
...     return self._cards[i]
...
>>>
>>> deck = Deck()
>>> len(deck)
52
>>> deck[0], deck[-1]
(Card(rank='2', suit='clubs'), Card(rank='A', suit='hearts'))
>>> deck[:3]
[Card(rank='2', suit='clubs'), Card(rank='3', suit='clubs'), Card(rank='4', suit='clubs')]

{{< /highlight >}}

> dunder 方法 len 的调用方式并不是作为对象方法调用，而通过一个通用方法 len ，这其实是基于性能考量，获取集合长度这个操作如此常见，以至于在对象中我们常常定义一个字段，而不是使用一个方法调用来获取。比如 Java 中 length 也是字段，而非方法调用。因此我们在实现的时候要和系统的设定保持一致。

因为可以基于 index 取值，当然也可以循环和排序

{{< highlight pycon >}}
>>> for card in deck[:3]:
...   print(card)
...
Card(rank='2', suit='clubs')
Card(rank='3', suit='clubs')
Card(rank='4', suit='clubs')
>>>
>>> it = iter(deck)
>>> it
<iterator object at 0x7f85efbd6710>
>>> next(it)
Card(rank='2', suit='clubs')
>>>
>>>
>>> SUITS
['clubs', 'diamonds', 'spades', 'hearts']
>>> SPADES_HIGH = sorted(SUITS)
>>> SPADES_HIGH
['clubs', 'diamonds', 'hearts', 'spades']
>>> def index(card):
...   """Card pos in spades high ordering"""
...   rank_value = RANKS.index(card.rank)
...   return rank_value * len(SPADES_HIGH) + SPADES_HIGH.index(card.suit)
...
>>> index(Card('2', 'clubs'))
0
>>> index(Card('A', 'spades'))
51
>>> sorted(deck, key=index)[:5]
[Card(rank='2', suit='clubs'), Card(rank='2', suit='diamonds'), Card(rank='2', suit='hearts'), Card(rank='2', suit='spades'), Card(rank='3', suit='clubs')]

{{< /highlight >}}

### 洗牌和动态补丁

抽取一张牌应该可以利用内置包 random

{{< highlight pycon >}}
>>> import random
>>> random.choice('ABCDE')
'E'
>>>
>>> random.choice(deck)
Card(rank='5', suit='hearts')
{{< /highlight >}}

random 包中也有 shuffle 操作，但应用到 Deck 却会出错:

{{< highlight pycon >}}
>>> num = list(range(5))
>>> random.shuffle(num)
>>> num
[4, 0, 2, 1, 3]
>>>
>>> try:
...   random.shuffle(deck)
... except TypeError as e:
...   print(repr(e))
...
TypeError("'Deck' object does not support item assignment",)

{{< /highlight >}}

这是因为 Deck 是不可变的集合，作为动态语言，python 当然支持在运行时对类定义做修改

{{< highlight pycon >}}
>>> def put(deck, position, card):
...   deck._cards[position] = card
...
>>> Deck.__setitem__ = put
>>>
>>> deck[:3]
[Card(rank='2', suit='clubs'), Card(rank='3', suit='clubs'), Card(rank='4', suit='clubs')]
>>> random.shuffle(deck)
>>> deck[:3]
[Card(rank='K', suit='hearts'), Card(rank='2', suit='diamonds'), Card(rank='2', suit='hearts')]

{{< /highlight >}}

### 操作符重载

总结 Deck 的定义:

{{< highlight python >}}
from itertools import chain

class Deck:
    def __init__(self, cards=CARDS):
        self._cards = list(cards)

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, i):
        if isinstance(i, slice):
            return Deck(self._cards[i])
        return self._cards[i]

    def __setitem__(self, i, card):
        self._cards[i] = card

    def __add__(self, other):
        #return Deck(list(self) + list(other))
        #return Deck(self._cards + list(other))
        return Deck(chain(self._cards, other))

    def deal(self, num_players):
        return [self[i::num_players] for i in range(num_players)]

{{< /highlight >}}

第一个改动是 dunder getitem 方法: 作为一个集合类，用户可以选择选取一打 Cards，而 Deck 作为 Cards 的容器，第一候选返回类型当然是一个 Deck 而非 List。
第二个改动是 dunder add 方法: 这是对 + 操作符重载的关键。选用最终 [chain 的方式](https://docs.python.org/3/library/itertools.html#itertools.chain)是从内存占用效率的考量。

{{< highlight pycon >}}
>>> deck = Deck()
>>> top3 = deck[:3]
>>> top3
<__main__.Deck object at 0x7f85efbcfe10>
>>> bottom3 = deck[-3:]
>>> top3 + bottom3
<__main__.Deck object at 0x7f85efbe5518>
>>> [ _ for _ in top3 + bottom3]
[Card(rank='2', suit='clubs'), Card(rank='3', suit='clubs'), Card(rank='4', suit='clubs'), Card(rank='Q', suit='hearts'), Card(rank='K', suit='hearts'), Card(rank='A', suit='hearts')]
>>> 
>>> random.shuffle(deck)
>>> a, b, c, d = deck.deal(4)
>>> len(a), len(b)
(13, 13)
>>> for card in c[:3]:
...   print(card)
... 
Card(rank='2', suit='diamonds')
Card(rank='5', suit='diamonds')
Card(rank='6', suit='diamonds')
{{< /highlight >}}

### 操作符重载之美

{{< highlight python >}}
import numpy as np
import matplotlib.pyplot as plt

v0 = 5
g = 9.81
t = np.linspace(0, 1, 1001)

y = v0 * t - g * t**2 / 2

plt.plot(t, y)
plt.xlabel('t (s)')
plt.ylabel('y (m)')
plt.show()
{{< /highlight >}}

{{< figure src="/img/blog/play-pythonic-cards/round-trip.png" title="Fall back" >}}

参考文档

> - Shahriar Tajbakhsh (2015-02-17) [Underscores in Python](https://shahriar.svbtle.com/underscores-in-python)
> - Mohit Sharma (2016-07-12) [Jupyter notebooks in blog](https://sharmamohit.com/post/jupyter-notebooks-in-blog/)
> - Nbconvert doc [Customizing nbconvert](http://nbconvert.readthedocs.io/en/latest/customizing.html)

封面图片来自 [Playing Arts](https://dribbble.com/shots/2112339-Playing-Arts) <a href="https://dribbble.com/creativemints"><i class="fa fa-dribbble" aria-hidden="true"></i> Mike | Creative Mints</a>
