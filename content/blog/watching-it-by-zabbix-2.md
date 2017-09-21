+++
date = "2017-08-24T22:24:51+08:00"
title = "Zabbix 自定义指标"
showonlyimage = false
image = "/img/blog/watching-it-by-zabbix-2/energy_system_icon_03.png"
draft = false
weight = 72
+++

定制监控指标的高阶操作和注意事项
<!--more-->

## 总述

监控产品的一个重要特性是一定要满足监控指标的定制化：比如现在 Docker 很火，那么如果有一台机器作为容器的宿主服务器，那么除了常规的监控指标，你还需要定制一些专用于监控这些容器的性能指标。

本文基于 ["Zabbix 综合监控"]({{< relref "blog/watching-it-by-zabbix-1.md" >}}) 的环境接续使用，并着重记录一下相关的操作实践。

首先是对虚拟机 D 做配置，按照 finid (2016-11-02) [How To Install and Use Docker on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-centos-7) 安装配置 Docker 服务及相关命令的试车。尤其是如果你下载 images 需要添加代理，请参考 Stackoverflow 上 [Cannot download Docker images behind a proxy](https://stackoverflow.com/a/28093517/4393386) 的解决方案。

## 试车

Christophe Labouisse (2014-11) [Simple Monitoring for Docker (Part I)](http://www.labouisse.com/how-to/2014/11/17/simple-monitoring-for-docker-part-1) 首先在容器宿主服务器上准备一个 shell 脚本，通过不同参数收集如下指标:

- 正在运行的容器数量
- 所有崩溃的容器数量(容器已停止且返回值非0)
- 所有容器数量

{{< highlight bash >}}
#!/bin/bash

function countContainers() {
  docker ps -q $1 | wc -l
}

function countCrashedContainers() {
  docker ps -a | grep -v -F 'Exited (0)' | grep -c -F 'Exited ('
}

TYPE=${1-all}

case $TYPE in
  running) COUNT_FUNCTION="countContainers"; shift;;
  crashed) COUNT_FUNCTION="countCrashedContainers"; shift;;
  all) COUNT_FUNCTION="countContainers -a"; shift;;
esac

$COUNT_FUNCTION

{{< /highlight >}}

类似的，我们还可以再准备一个收集宿主服务器上镜像数量和 [dangling 的镜像](https://www.projectatomic.io/blog/2015/07/what-are-docker-none-none-images/)数量

> 唯一需要注意的是 shell 脚本直接调用 docker 命令，为了避免 sudo 配置，可能需要将 zabbix 用户加入到 docker 用户组——实在记不清我实验的时候是否做了相关步骤。

## 原理

Docker Inc 的官博 Jérôme Petazzoni (2013-10-08) [Gathering LXC and Docker Containers Metrics](https://blog.docker.com/2013/10/gathering-lxc-docker-containers-metrics/) 值得阅读。Control groups 从 Redhat 6/7 被引入，可以通过 df 查看是否存在 /cgroup 或 /sys/fs/cgroup 挂载点来判决。



## 集成

[Simple Monitoring for Docker (Part II)](http://www.labouisse.com/how-to/2014/11/18/simple-monitoring-for-docker-part-2)

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
