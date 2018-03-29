+++
date = "2018-03-22T20:55:21+08:00"
title = "获取OSM街道信息"
showonlyimage = false
image = "/img/blog/download-street-network/map.png"
draft = false
weight = 605
+++

获取开源地图行政区域和接到数据
<!--more-->

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

- Matthew Bilyeu (2018-01-21) [Python environment with Pipenv, Jupyter, and EIN](https://matthewbilyeu.com/blog/python-environment-with-pipenv-jupyter-and-ein/)


封面图片来自 [A Map](https://dribbble.com/shots/2207264-A-Map) <a href="https://dribbble.com/JustinMezzell"><i class="fa fa-dribbble" aria-hidden="true"></i> Justin Mezzell</a>
