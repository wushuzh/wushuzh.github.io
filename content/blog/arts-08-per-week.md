+++
date = "2018-08-25T19:33:04+08:00"
title = "ARTS 2018w34"
showonlyimage = false
image = "/img/blog/arts-08-per-week/neacademia.jpg"
draft = false
weight = 1834
tags = ["ARTS"]
+++

翻译石碑 vs 通天塔，以及避免无效争论
<!--more-->

## Algorithm

本周 leetcode 题目 [445. Add Two Numbers II](https://leetcode.com/problems/add-two-numbers-ii/description/) 和上周题目 [2. Add Two Numbers](https://leetcode.com/problems/add-two-numbers/description/) 类似。

不同之处是上回链表存储时，要求头部为最低位，而这回要求头部为最高位——记得加法要从最低位加起，而且本题特别要求不能对原链表做逆转操作。

> Input: (7 -> 2 -> 4 -> 3) + (5 -> 6 -> 4)  
> Output: 7 -> 8 -> 0 -> 7

- 思路一：通过计算两个链表长度，将对齐位做加法存放在结果链表中，然后再处理进位——使用双指针标记需要进位的起始点，这是因为要考虑到连续进位的情况;
- 思路二：借用“类似 stack”的数据结构来转存数据，从而迂回地实现翻转(`reverse`)，继而从低位相加；
  - Dan Bader [linked lists in python](https://dbader.org/blog/python-linked-list)
- 对于测试断言，可为 NodeList 类添加 `__eq__` 方法，以检测全链的元素是否相等:
  - [Quora解答](http://qr.ae/TUN4hW)： 其中 [car、cdr](https://en.wikipedia.org/wiki/CAR_and_CDR)、[列表构造函数 cons](https://en.wikipedia.org/wiki/Cons) 是 LISP 的经典函数。
  - SO [Elegant ways to support equivalence (“equality”) in Python classes](https://stackoverflow.com/a/25176504/4393386)
- 对于调试打印，可为 NodeList 类添加 `__repr__`、`__str__`、`__iter__` 方法，延伸阅读:
  - Dan Bader [Python String Conversion 101: Why Every Class Needs a “repr”](https://dbader.org/blog/python-repr-vs-str) 
  - SO [Difference between `__str__` and `__repr__`?](https://stackoverflow.com/a/2626364/4393386)

## Review  

Sander van Vugt 的 Linux Under the Hood 关于网络的笔记——讲的不深入，顶多算个入门。

{{< figure src="/img/blog/arts-08-per-week/intro-linux-network.jpg" title="Linux network intro" >}}

## Tip

之前我看 [hypothesis 速成文档](https://hypothesis.readthedocs.io/en/latest/quickstart.html)时，其中[“游程编码问题”](https://en.wikipedia.org/wiki/Run-length_encoding)的解答直接引用了 Rosetta Code 上的 [python 代码](http://rosettacode.org/wiki/Run-length_encoding#Python)。

> [罗塞塔石碑 Rosetta Stone](https://en.wikipedia.org/wiki/Rosetta_Stone) 是一块刻有古埃及法老诏书的石碑，它使用三种语言(古埃及象形文，埃及草书，古希腊文)记录了同一内容，这成为日后研究和解读古埃及文的重要依据。基本上它和传说中的[巴别塔](https://en.wikipedia.org/wiki/Tower_of_Babel)的寓意完全相反。

而[“罗塞塔代码”](https://en.wikipedia.org/wiki/Rosetta_Code) 借用了这个概念，它针对近千个问题用七百多种编程语言做了示例性的实现。我以前只见过一个用不同的 MV* 框架实现同一个 Todo 应用的网站: [TodoMVC](http://todomvc.com/)，这回翻到了下面这个[列表](http://rosettacode.org/wiki/Help:Similar_Sites)——罗列了不少类似的网站，虽少部分无法访问，但仍不失是一个挺好的学习参考资料。

其实 leetcode 是很有条件做类似的网站的。

## Share

可能不少人都有过“举世皆浊我独清，众人皆醉我独醒”感受，这两周看了先是看到了王川的文章，感觉一下子点醒了我——以前那些争论真是太不值了。

 (2018-08-28) [《王川:不要打扰别人心中的故事》](http://wangchuan.blog.caixin.com/archives/187047)

1. 大众平时都在传谣——因为传播又快又广的信息都会采用叙事模式，中间我们会对信息做压缩及脑补；
2. 从这些扭曲的信息中将会得到诡异的结论。更有有些行动派用其直到行动，但只要不全投加杠杆，他们会失败的；
3. 信息时代，各种谣言会在适当时候迅速消失，被新的谣言取代；
4. 要大量收集资料，冷静分析，远离外围谣言，避免被碾压裹挟；

## 拓展阅读

> - 陈皓 (2016-07-11) [为什么我不在微信公众号上写文章](https://coolshell.cn/articles/17391.html)
> - 微信公号 fstalk (2017-03-05)《傅盛认知三部曲之一：所谓成长就是认知升级》

封面图片来自 [Neacademia Broadsheet](https://dribbble.com/shots/2090965-Neacademia-Broadsheet) <a href="https://dribbble.com/rosettatype"><i class="fa fa-dribbble" aria-hidden="true"></i> Rosetta Type Foundry</a>