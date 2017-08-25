+++
date = "2017-08-24T22:24:51+08:00"
title = "Zabbix 自定义指标"
showonlyimage = false
image = "/img/blog/watching-it-by-zabbix-2/energy_system_icon_03.png"
draft = true
weight = 72
+++

定制监控指标的高阶操作和注意事项
<!--more-->

## 总述

监控产品的一个重要特性是一定要满足监控指标的定制化：比如现在 Docker 很火，那么如果有一台机器作为容器的宿主服务器，那么除了常规的监控指标，你还需要定制一些专用于监控这些容器的性能指标。

本文的环境接续前面的 ["Zabbix 综合监控"]({{< relref "blog/watching-it-by-zabbix-1.md" >}})，并着重记录一下相关的操作实践。

首先是对虚拟机 D 做配置，按照 finid (2016-11-02) [How To Install and Use Docker on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-centos-7) 配置 Docker 运行环境。

## 原理

Christophe Labouisse (2014-11) [Simple Monitoring for Docker (Part I)](http://www.labouisse.com/how-to/2014/11/17/simple-monitoring-for-docker-part-1)
和 [Simple Monitoring for Docker (Part II)](http://www.labouisse.com/how-to/2014/11/18/simple-monitoring-for-docker-part-2)

配置 Docker 代理
https://stackoverflow.com/questions/23111631/cannot-download-docker-images-behind-a-proxy

通过 EPEL RPM 源安装 python-pip

{{< highlight console >}}

pip install docker-py

{{< /highlight >}}

## 实践

https://www.zabbix.com/documentation/3.0/manual/config/items/userparameters/extending_agent

另外 agentless 应该如何做？

参考文档

> - Justin Ellingwood tutorial Series [The Docker Ecosystem] (https://www.digitalocean.com/community/tutorial_series/the-docker-ecosystem)
> - https://www.digitalocean.com/community/tags/docker?type=tutorials

封面图片来自 [Energy System Icon 03](https://dribbble.com/shots/1054192-Energy-System-Icon-03) <a href="https://dribbble.com/Kingyo"><i class="fa fa-dribbble" aria-hidden="true"></i> Kingyo</a>
