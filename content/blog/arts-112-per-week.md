+++
date = "2020-08-03T14:07:42+08:00"
title = "ARTS 2020w31"
image = "/img/blog/arts-112-per-week/"
draft = true
weight = 2031
tags = ["ARTS"]
+++


<!--more-->

## Algorithm


## Review

## Tip


## Share

在无法连接互联网的自有实验室，搭建一个 kubernetes 实验集群的技术方案。

基本思路是通过一个能上网的服务器下载各种需要的软件包，传输到隔离的实验室内的服务器上，按各自角色执行安装和配置工作。

需要的工作：

- 能上网的服务器利用 vagrant 可以获得和离线服务器相同的OS版本；
- 对安装软件进行组织和版本控制；
- 利用脚本高效向各个实验室的服务器传输各自需要的软件安装包；
- 利用 pxe kickstart 等方式为实验室服务器安装系统；
- 为集群增加一个新的工作节点；

为 kubernetes 集群增加一个工作节点


封面图片来自 [](d) <a href=""><i class="fa fa-dribbble" aria-hidden="true"></i> </a>