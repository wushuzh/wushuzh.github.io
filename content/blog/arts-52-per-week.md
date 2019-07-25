+++
date = "2019-06-17T14:14:47+08:00"
title = "ARTS 2019w25"
image = "/img/blog/arts-52-per-week/alone.webp"
draft = true
weight = 1925
tags = ["ARTS", "Union-Find"]
+++

专注既定目标，屏蔽吵杂干扰
<!--more-->

## Algorithm


## Review

Steve Yegge 的老帖子 (2008-3-12) [Get that job at Google](https://steve-yegge.blogspot.com/2008/03/get-that-job-at-google.html) 给出了软件工程师准备技术型公司的工作面试的一些建议。

首先心理上的准备：

- 基于现有面试手段上的局限，你没通过不代表你没能力；
- 如果出现自己、面试官状态不佳、水逆等偶然情况，别放心上，半年、一年以后再去试试；

面试前的热身：

- 熟悉白板编程，其他形式都不在话下；
- 学习一本数据结构+算法书，目的是理解各种适用场景；(推荐  Steven Skiena The Algorithm Design Manual)
- 找朋友做模拟面试；

面试时的非技术建议：

- 面试心态要谦虚开放，专注在给定题目上，动手编码前一定要先沟通思路。甚至要求提示，确认当前的思路是否正确；
- 注意保持节奏紧凑，面试题可能不止一道；
- 自己准备白板笔和板擦以防万一；

面试时的技术建议：

- 要会分析算法复杂度；
- 要会两种以上的 n*long(n) 的排序方法；
- 必须会实现 hashtable；
- 要至少会[二叉树](https://en.wikipedia.org/wiki/Binary_tree)、[M叉树](https://en.wikipedia.org/wiki/M-ary_tree)、[前缀树](https://en.wikipedia.org/wiki/Trie)，至少了解一种[平衡树](https://en.wikipedia.org/wiki/Self-balancing_binary_search_tree)，宽度/深度搜索，[各种遍历顺序](https://www.geeksforgeeks.org/tree-traversals-inorder-preorder-and-postorder/)；
- 图非常重要：[各种表示方法](https://www.khanacademy.org/computing/computer-science/algorithms/graph-representation/a/representing-graphs)，基本遍历方法：宽度/深度优先

数学：概率、排列组合、基础的离散数学知识；  
操作系统：线程、进程所需要的资源，互斥锁，信号量，推荐  Doug Lea Concurrent Programming in Java  
编程语言：C++ 和 Java 至少熟悉一种；  

## Tip

Redhat 上个月[发布了 Universal Base Image](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image) ：

- 不但可运行在 Redhat 7/8 平台，也可以用在其他任何兼容 OCI 的容器平台；
- 通用型的 baseimage 有 ubi-minimal，ubi，ubi-init；
- 含特定语言运行时的 baseimage 支持 nodejs, python, ruby；
- Redhat 会在提供一个专有的 yum repo ，以便客户能自行及时、按需更新软件和安全补丁；

base image 是如何构建的 ？简单介绍见 fatherlinux Jun 17, 2019 [Where’s The Red Hat Universal Base Image Dockerfile?](http://crunchtools.com/ubi-build/)

{{< figure src="/img/blog/arts-52-per-week/redhat-base-img.jpg" title="REDHAT UBI" >}}

## Share

本周读了《深度工作》，之所以读这本书，是感觉自己总是很忙的样子，但是回头看，其实真正的成果不多。我希望更好的提升做事效率。

本书作者应该是麻省理工的教授，工作领域也是计算机，分享总结了自己的一些做法。我快速读完，列出一些对我有用的部分：

- 首先从思想上要认可“深度工作”，承认进入“心流”状态可以为自己带来其他方式无法替代的价值；
- 要从日常工作习惯上要做出主动改变，让“深度工作”成为常规项目，最大可能地避免引入过重的决策和心理负担；
- “注意力”是个人最宝贵的资源，而注意力的可持续时间是可以通过训练提高的；
- 最大程度避免刷社交 App，有限地访问邮箱，你也许不会错过什么真正有价值的事情；
- 要勇敢而主动地按自己的“深度工作”方式引导事情走向，有技巧地面对日常肤浅工作；
- 有意识地记录回顾，并尝试部分地将个人目标公开，要学会利用社会、同辈压力鞭策自己达成目标；
- 永远只做最重要的事情；

我的一个感想是，我之前做了太多的“肤浅工作”，我同时猜想这个世界肯定有很大比例的人也都是这样工作、生活着——而能想清楚自己的目标，拥有一个坚定的处事原则的人数比例并不高。对于暂时想不清楚的人自己要做、能做什么的人，最好还是在那些有清晰远景的人的指导下开展学习工作，虽然这样做的效率可能不会太高。

如果你愿意，当然可以通过观察学习到别人在一步步拆解和实现他们的终极目标的过程中，抓住了什么，又放弃了哪些？而在这个过程中如果你偶然发现了自己的目标，并且有强烈的愿望去实现，就可以逐步地，有计划地尝试去掌控自己的人生。但如果什么目标，就随着大家继续生活。

不同的生活方式其实没有什么对错，只有适合自己与否。

封面图片来自 [Keep hoping](https://dribbble.com/shots/4453652-Keep-hoping) <a href="https://dribbble.com/jiayang"><i class="fa fa-dribbble" aria-hidden="true"></i> Jiayang</a>