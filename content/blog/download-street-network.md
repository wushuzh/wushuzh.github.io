+++
date = "2018-03-22T20:55:21+08:00"
title = "获取OSM街道信息"
showonlyimage = false
image = "/img/blog/download-street-network/map.png"
draft = true
weight = 605
tags = ["python", "map"]
+++

未完草稿——获取开源地图行政区域和接到数据
<!--more-->

Geo 字根为地球，graphy 意为描述。按照 Dr. Heath Robinson 的宏大说法，地理学的核心可以描述为对我们的世界综合而全面的理解。(A comprehensive and holistic understanding of the entire world)。而其最常见的问题都是以 Where 开头，相应的分析分为两个层次: 
  1. 对事实收集、描述 (What is it like ?)
  2. 在上下文环境下，对上面事实的解释/分析、推断/预测 

以公元前希腊全才人物[埃拉托斯特尼](https://en.wikipedia.org/wiki/Eratosthenes)为例，他测算了分别位于南北两城的方尖塔、深井、和城距，推测地球是圆的，并结合几何和三角学，计算出地球周长。(2%误差)

## 经纬度

Introduction to GIS I: Fundamentals and Mapping 8-13

首先安装

cower -d spatialindex
cd osmnx_demo
pipenv install jupyter osmnx
pipenv run jupyter notebook

- 获取行政区域的shapefiles
- 获取街道数据:可以获得一定区域内的机动车道，自行车道，步行道等
- 简化街道拓扑
- 保存街道网络
- 对网络做计算分析





参考文档

Geoff Boeing 
- [OSMnx: Python for Street Networks](http://geoffboeing.com/2016/11/osmnx-python-street-networks/)
- [Examples demonstrating the usage of OSMnx](https://github.com/gboeing/osmnx-examples)
Emacsen
- (2018-01-04) [Why the world needs OpenStreetMap](https://blog.emacsen.net/blog/2014/01/04/why-the-world-needs-openstreetmap/)
- (2018-02-16) [Why OpenStreetMap is in Serious Trouble](https://blog.emacsen.net/blog/2018/02/16/osm-is-in-trouble/)

- Dr. Heath Robinson [What is Geoprahy ?](https://www.youtube.com/playlist?list=PLRNNjIk9ArAovzZ_6c5TSeggFBhw-gxLE)
- Matthew Bilyeu (2018-01-21) [Python environment with Pipenv, Jupyter, and EIN](https://matthewbilyeu.com/blog/python-environment-with-pipenv-jupyter-and-ein/)
- Department of Geosciences & Geography, University of Helsinki [Automating GIS processes course](https://automating-gis-processes.github.io/2017/)


封面图片来自 [A Map](https://dribbble.com/shots/2207264-A-Map) <a href="https://dribbble.com/JustinMezzell"><i class="fa fa-dribbble" aria-hidden="true"></i> Justin Mezzell</a>
