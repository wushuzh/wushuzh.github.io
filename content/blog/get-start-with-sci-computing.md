+++
date = "2017-08-03T20:48:43+08:00"
title = "科学计算"
showonlyimage = false
image = "/img/blog/get-start-with-sci-computing/lab-boy.png"
topImage =  "/img/blog/get-start-with-sci-computing/lab-boy.gif"
draft = false
weight = 51
+++

用 Fedora 定制版 Python 快速上手科学计算
<!--more-->

最近看了 Jake Vanderplas 在 PyCon 2017 的讲演 [The unexpected Effectiveness of Python in Science](https://youtu.be/ZyjCqQEUa8o) 概要性的了解了一下科学计算，顺带发现一些有用的资源，记录在这里，待今后一一学习。

jakevdp 不但是天文学家，而且也是一些重量级 python 科学计算模块(比如 Scikit-Learn， Scipy，AstroML [等等](https://staff.washington.edu/jakevdp/projects.html))的主要贡献者，他还是下面两本书的作者，并将全书放在网上供学习者免费阅读

- [The Python Data Science Handbook](https://jakevdp.github.io/PythonDataScienceHandbook/)
- [A Whirlwind Tour of Python](https://jakevdp.github.io/WhirlwindTourOfPython/)

一般人想象中的天文学家当然都是在天文台工作:在那些月黑风高的深夜，开启一台巨大的天文望远镜，通过调整各种用于微调、定焦的复杂旋钮，同时目不转睛地盯着一小片天空试图研究某一个遥远的星系。

但事实上，相当一部分天文学家都不会操作甚至从来没有触碰过这些高大上的巨型望远镜:

- 根据接受光的频率不同，带有目镜能用肉眼观测的望远镜已不是专业科研人员依靠的唯一工具;
- 如果是空间望远镜压根没在地球上，比如哈勃、开普勒分别在低地球轨道、日心轨道上运行;
- 比如坐落于贵州的全球最大单一口经 FAST 望远镜，如此精密、贵重的设备需要专业运维人员才能触碰，我猜应该是全世界的天文学家只能向其提交观测需求，排队等待，最后获得数据，不能真正参与和观测直接相关的操作

他们直接从数据库中调取相应区域的数据集，比方说天空坐标，观测区域内各个恒星行星轨道，观测时各种光学参数，仪器本身需修正的参数等，然后综合使用统计分析、机器学习等方法提出假设，验证结论。比如著名的 [Kepler Mission](https://en.wikipedia.org/wiki/Kepler_(spacecraft)) 在其 [github 主页](https://github.com/KeplerGO) 上开源其数据分析的工具——并使用 python 作为其官方分析语言。

我们一般从新闻上看到的宜居星系是这样的:

{{< figure src="/img/blog/get-start-with-sci-computing/img.png" title="Imaginery" >}}

而天文学家得到的原始数据是这样的:

{{< figure src="/img/blog/get-start-with-sci-computing/real.png" title="Real Data" >}}

封面图片来自 [Scientist](https://dribbble.com/shots/2374402-Scientist) <a href="https://dribbble.com/aga-kozak"><i class="fa fa-dribbble" aria-hidden="true"></i> Aga Kozak</a>  


https://youtu.be/FytuB8nFHPQ

https://labs.fedoraproject.org/en/python-classroom/

SciPy 2017
https://www.youtube.com/playlist?list=PLYx7XA2nY5GfdAFycPLBdUDOUtdQIVoMf
