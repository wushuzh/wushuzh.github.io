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

### 准备数据库

{{< highlight console >}}
$ pipenv install bokeh pandas psycopg2 --three
$ mkdir -p project/db
$ touch project/db/create.sql
$ touch project/db/Dockerfile
# prepare content for db

$ docker build -t stock-db .
# hava a try
$ docker run --rm -d -e POSTGRES_PASSWORD=postgres stock-db
<hash-id>
# note it takes a while for db full startup, so it need to 
#   add check health when we turn it into a service 

$ docker exec -ti <hash-id> psql -U postgres
{{< /highlight >}}

{{< highlight psql >}}
psql (10.0)
Type "help" for help.

postgres=# \c stocks
You are now connected to database "stocks" as user "postgres".
postgres=# \dt
              List of relations
 Schema |       Name       | Type  |  Owner   
--------+------------------+-------+----------
 public | stock_highlow    | table | postgres
 public | stock_image_urls | table | postgres
 public | stock_prices     | table | postgres
(3 rows)

postgres=# \q

{{< /highlight >}}

### 抓取程序

通过股票代码获得其价格使用了 IEX API，按照原教程，将其封装为一个通用的接口，使得我们保有将其替换为其他 API 的能力。具体做法首先利用 abc 模块创建一个抽象类 StockFetcher 并定义好抓取不同信息的接口。

之后就可以编写 IEXStockFetcher 类了。 通过 urllib.request ，参考 [IEX API](https://iextrading.com/developer/docs) URL 格式和返回 json 数据结构，返回当前价，股票标志，年内价格区间。注意:

1. urllib 会自动应用系统的网络代理。
2. 原教程的无限重试 try except 会产生 stack overflow ，我尝试了其他几种的重试方式:
    - [循环休眠跳出](https://github.com/wushuzh/stockstreamer/commit/995bfa2c73fac357c270102c59092695912b6ce7)
    - [函数式调用](https://github.com/wushuzh/stockstreamer/commit/2f6aef5011b31a7ce2b223d45f7f9bb9663045c4)
    - [借助 retrying 模块](https://github.com/wushuzh/stockstreamer/commit/54ff0d2474ec20583b91d05fd7f3a0b039745ea4)

{{< highlight console >}}
Fatal Python error: Cannot recover from stack overflow.

Current thread 0x00007ff742c72540 (most recent call first):
  ...
  File "~/somepath/stockstreamer/project/data_fetcher.py", line xx in fetchPrice
  ...
Aborted (core dumped)

{{< /highlight >}}

完成了抓取解析数据的核心功能，下面开始考虑怎么做的更有效率。一个基本需求是获取一系列的股票的信息，但是使用普通循环的问题是每个查询请求在获得结果前都会阻塞余下的查询。

- 这个问题可以通过引入线程解决: 即等待网络结果的时候，线程调度器自动切换到各个线程上下文执行相应工作。
- 考虑到从线程获得返回值比较麻烦，所有这里采用的方法是就向每个线程传入同一个 dict 集成所有的结果。
- functools.partial 使得目标函数可以直接通过实例方法引用(即便没有将 IEXStockFetcher 实例化)

这部分代码见 [5131772](https://github.com/wushuzh/stockstreamer/commit/513177294464dcf6f8f3a7a214efb927ad61c15a)

> TODO: 貌似 IEX API 只能获得美股情报，以后找找是否有适用于全球(尤其中国、香港)的 API

### 存储数据

本节是将抓好的数据入库表，构建基类 Manager 调用 Fetcher 并入库，和原教程不同，我打算后续尝试更换数据库——换成 mariadb 的意义可能不大，甚至子类 PostgreSQLStockManager 基本可以改造为通吃的方式(根据 dburl 判定使用连接驱动？)，但如果尝试换成 Mongodb 或 Cassandra 可能变得更为简单。

针对每个表:

1. 首先获取所有股票代码的数据
2. 将每股的数据逐一拆出入库，然后休眠一定时间无限重复

### 展示程序


参考文档

> - RealPython [Flask Bokeh Example](https://github.com/realpython/flask-bokeh-example/)
> - Ethan Cerami (2017-04-13) [Creating Interactive Bokeh Applications with Flask](http://biobits.org/bokeh-flask.html)
> - Dan Bader [Abstract Base Classes in Python](https://dbader.org/blog/abstract-base-classes-in-python)
> - Quora [Are there any open APIs for free, accurate, real-time financial market data?](https://www.quora.com/Are-there-any-open-APIs-for-free-accurate-real-time-financial-market-data)

{{< speakerdeck a2d86983ff634ac3871ad4e5a308a67b >}}

封面图片来自 [Free Bokeh Backgrounds](https://dribbble.com/shots/2056186-Free-Bokeh-Backgrounds) <a href="https://dribbble.com/Graphicsoulz"><i class="fa fa-dribbble" aria-hidden="true"></i> Graphicsoulz</a>
