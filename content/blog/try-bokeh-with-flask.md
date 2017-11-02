+++
date = "2017-10-31T20:59:31+08:00"
title = "Bokeh 数据可视化"
showonlyimage = false
image = "/img/blog/try-bokeh-with-flask/bokeh-bgs.png"
topImage = "/img/blog/try-bokeh-with-flask/bokeh-bgs.gif"
draft = false
weight = 110
+++

使用 Python 生成交互式图表
<!--more-->

## Python 可视化

Python 可视化是一个很大的话题。对其总结概括的最为全面的是 PyCon 2017 Jake VanderPlas 的主题讲演 [The Python Visualization Landscape](https://youtu.be/FytuB8nFHPQ) 重点讨论了

- 团结在 matplotlib 为核心的领域模块
- 围绕在 javascript 周围的各种 json 序列化方案
- 代表着可视化未来方向的 Vega 和 Altair 

<img alt="py-vis" src="/img/blog/try-bokeh-with-flask/py-vis.png" class="img-responsive">

## Bokeh 入門

这部分是对 Matt Makai (2017-07-30) [Responsive Bar Charts with Bokeh, Flask and Python 3](https://www.fullstackpython.com/blog/responsive-bar-charts-bokeh-flask-python-3.html) 的实践版。基本步骤是

- 在虚拟环境中搭建脚手架
- 模拟数据并 DF 化
- 将数据做最基础的展示
- 添加鼠标悬浮时的网页交互配置
- 微调各种可显示元素

我实践中和原文的主要的区别有

- 使用当下最新的 bokeh 版本 0.12.10
- 模拟数据时对 x 数值字符化，否则 FactorRange 报错
- 实现上拆分基本画图和微调到多步
- 借助接口填充 html 模板中 js 和 css 部分

具体代码见 [github](https://github.com/wushuzh/barchart)

### 基础架构

{{< highlight console >}}
$ pipenv --three
$ pipenv install flask bokeh pandas
$ mkdir -p barchart/templates
$ tocuh bootstrap.sh
$ touch barchart/__init__.py
$ touch barchart/app.py
$ touch barchart/templates/chart.html
...
$ ./bootstrap.sh &
$ curl http://localhost:5000/42/
<!DOCTYPE html>
<html>
    <head>
        <title>Bar charts with Bokeh!</title>
    </head>
    <body>
        <h1>Bugs found over the past 42 days</h1>
    </body>
</html>

{{< /highlight>}}

### 显示设定

学习新知识，最重要的是知晓新领域中的重要概念和术语，这对于理解工具的最佳使用方式，乃至定制高阶解决方案至关重要。Bokeh 用户手册专門有一章介绍了[术语和接口](https://bokeh.pydata.org/en/latest/docs/user_guide/concepts.html)，应该反复阅读。

- Bokeh 参考手册中 [models 章节](https://bokeh.pydata.org/en/latest/docs/reference/models.html) 包含了所有底层接口，全面负责作图和小工具配置
- plotting 包含了高层接口，创建各种 glyphs 最终组装(add_glygh)成最终的 plot

## Bokeh 服务器

[Bokeh 服务器](https://bokeh.pydata.org/en/latest/docs/user_guide/server.html)是一个可选组件，引入它可以获得获得更多交互功能、连接后端做实时数据更新，甚至使用各种相关事件的回调什么的。

找到一个入门案例 Alan "AJ" Pryor (2017-09-24) [Building an interactive, data-driven web-application using Amazon EC2 and Python](http://alanpryorjr.com/2017-09-24-stockstreamer/) 原文创建的一个应用的功能是抓取股票价格存入数据库，并将最新更新数据在前端展示。应用框架如下:

<img alt="stocks-app" src="/img/blog/try-bokeh-with-flask/stocks-app.png" class="img-responsive">

我的准备在自己的平台实践一下，预计和原文相比主要不同为:

- 不使用 AWS ，但会做容器化封装
- 尝试更换数据库 mariadb

{{< highlight console >}}
$ pipenv install bokeh pandas psycopg2 --three
$ mkdir -p project/db
$ touch project/db/create.sql
$ touch project/db/Dockerfile

{{< /highlight >}}
参考文档

> - RealPython [Flask Bokeh Example](https://github.com/realpython/flask-bokeh-example/)
> - Ethan Cerami (2017-04-13) [Creating Interactive Bokeh Applications with Flask](http://biobits.org/bokeh-flask.html)

{{< speakerdeck a2d86983ff634ac3871ad4e5a308a67b >}}

封面图片来自 [Free Bokeh Backgrounds](https://dribbble.com/shots/2056186-Free-Bokeh-Backgrounds) <a href="https://dribbble.com/Graphicsoulz"><i class="fa fa-dribbble" aria-hidden="true"></i> Graphicsoulz</a>
