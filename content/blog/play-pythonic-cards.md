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

Luciano Ramalho著有 fluent python 一书。最近看了他在 PyBay 2017 的一次分享，内容都在 [Jupyter Notebook](https://github.com/fluentpython/think-like-a-pythonista/blob/master/think-like-a-pythonista.ipynb) 上。

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

### 轻量级 tuple 类
 
通过 collections.namedtuple 可用来定义集合中的一个元素类型，他们不像标准类那么“重”，但却能增加不少类对象的功能。

假设定义每张扑克牌，
- 第一个参数，类名 Card
- 第二个参数，定义对象属性/字段: 这里当然是牌值和花色，放在一个字符串中并用空格/逗号分割即可，如'rank suit'

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

list comprehensive

Deck: immutable seq by implement dunder methods 
note: list converter during initilization

special len method, also in length is not method call but property
the answer is performance reason: this operation is so common a dedicated field is defined for this
when you define your own class, be consistent as well



https://github.com/fluentpython/think-like-a-pythonista/blob/master/think-like-a-pythonista.ipynb
https://youtu.be/M4gPxbo6G6k

https://www.cheatography.com/weidadeyue/cheat-sheets/jupyter-notebook/

https://shahriar.svbtle.com/underscores-in-python


{{< fullgist wushuzh 78474a8546aff563f8fbd6f8213f75a8 8000>}}

封面图片来自 [Playing Arts](https://dribbble.com/shots/2112339-Playing-Arts) <a href="https://dribbble.com/creativemints"><i class="fa fa-dribbble" aria-hidden="true"></i> Mike | Creative Mints</a>
