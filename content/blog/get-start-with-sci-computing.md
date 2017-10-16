+++
date = "2017-08-03T20:48:43+08:00"
title = "上手科学计算"
showonlyimage = false
image = "/img/blog/get-start-with-sci-computing/lab-boy.png"
topImage =  "/img/blog/get-start-with-sci-computing/lab-boy.gif"
draft = false
weight = 51
+++

使用 Fedora 定制版也能快速上手科学计算
<!--more-->

## 星际穿越

最近看了 Jake Vanderplas 在 PyCon 2017 的讲演视频 [The unexpected Effectiveness of Python in Science](https://youtu.be/ZyjCqQEUa8o) 概要性的了解了一下 Python 的科学计算，顺带发现一些有用的资源，记录在这里，待今后一一学习。

jakevdp 不但是天文学家，而且也是一些重量级 python 科学计算模块(比如 Scikit-Learn， Scipy，AstroML [等等](https://staff.washington.edu/jakevdp/projects.html))的主要贡献者，他还是下面两本书的作者，并将全书放在网上供学习者免费阅读

- [The Python Data Science Handbook](https://jakevdp.github.io/PythonDataScienceHandbook/)
- [A Whirlwind Tour of Python](https://jakevdp.github.io/WhirlwindTourOfPython/)

一般人想象中的天文学家当然都是在天文台工作:在那些月黑风高的深夜，开启一台巨大的天文望远镜，通过调整各种用于微调、定焦的复杂旋钮，同时目不转睛地盯着一小片天空试图研究某一个遥远的星系。

但事实上，相当一部分天文学家都不会操作甚至从来没有触碰过这些高大上的巨型望远镜:

- 根据接受光的频谱不同，装配有目镜能用肉眼观测的望远镜**不再是**天文学研究的唯一工具——现代天文学不仅仅要知晓存在，还要通过谱线确定行星的主要组成物质;
- 如果是空间望远镜——压根不是地面观测台，比如哈勃、开普勒就分别在低地球轨道、日心轨道上运行;
- 比如坐落于贵州的全球最大单一口经 FAST 望远镜，如此精密、贵重的设备需要专业运维人员才能触碰，我猜应该是全世界的天文学家只能向其提交观测需求，排队等待，最后获得数据，不能真正参与和观测直接相关的操作

他们直接从数据库中调取相应区域的数据集，比方说天空坐标，观测区域内各个恒星行星轨道，观测时各种光学参数，仪器本身需修正的参数等，然后综合使用统计分析、机器学习等方法提出假设，验证结论。比如著名的 [Kepler Mission](https://en.wikipedia.org/wiki/Kepler_(spacecraft)) 在其 [github 主页](https://github.com/KeplerGO) 上开源其数据分析的工具——并使用 python 作为其官方分析语言。

我们一般从新闻上看到的宜居星系是这样的:

<img alt="Imaginery" src="/img/blog/get-start-with-sci-computing/img.png" class="img-responsive">

而天文学家得到的原始数据是这样的:

<img alt="Real Data" src="/img/blog/get-start-with-sci-computing/real.png" class="img-responsive">

这些数据在分析前需要清洗，去噪，应对测量引入误差，对无法修复的故障进行补救，而且随着各国在航天领域的更多的关注和投入，需要研究的数据数量呈几何级数爆炸性增长。

## 科学栈

<img alt="Real Data" src="/img/blog/get-start-with-sci-computing/sci-stack.png" class="img-responsive">

这个图高屋建瓴的将当下科学计算领域的主流的 python 变种、模块、工具做了总结，每个都值得学习了解。因为科学研究必须要站在巨人的肩膀上。

Vanderplas 将 python 在科学领域大放异彩的原因归结如下:

- 首先是胶水语言
- 内置电池外加无限扩展包
- 科学探究需要简单易用，动态调整
- 严肃学术要求的科学气质(ethos)

python 很好地完成了科学研究的可重复性要求:“一个包含计算结果的文章只能叫做广告，而非学术。真正的学术要求完整的软件、代码和数据，并能产生你所声称的结果。”

> 比如刚刚获得诺贝尔奖的 LIGO 引力波团队不但发表了论文，而且发布了能重现其结果的 [Jupyter Notebook](https://losc.ligo.org/s/events/GW150914/GW150914_tutorial.html) 。

参考文档

> - Jake VanderPlas (2017-05-19) Slides of
    - [The Unexpected Effectiveness of Python in Science](https://speakerdeck.com/jakevdp/the-unexpected-effectiveness-of-python-in-science)
    - [The Python Visualization Landscape](https://speakerdeck.com/jakevdp/pythons-visualization-landscape-pycon-2017)
> - [The Fedora Python Classroom Lab](https://labs.fedoraproject.org/en/python-classroom/)
> - SciPy 2017 on [Youtube](https://www.youtube.com/playlist?list=PLYx7XA2nY5GfdAFycPLBdUDOUtdQIVoMf)

封面图片来自 [Scientist](https://dribbble.com/shots/2374402-Scientist) <a href="https://dribbble.com/aga-kozak"><i class="fa fa-dribbble" aria-hidden="true"></i> Aga Kozak</a>  
