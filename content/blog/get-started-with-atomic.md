+++
date = "2017-07-24T14:35:02+08:00"
title = "Atomic 试用"
showonlyimage = false
image = "/img/blog/setup-a-container-host-env/redhat-docker.png"
draft = true
weight = 51
tags = ["virtualization"]
+++

Atomic 服务器的维护和 Docker 运行
<!--more-->

## Atomic

Atomic 主要为 Docker 提供运行环境。其上和 Docker 相关的软件、系统服务都已预装配置完毕，直接达到开袋即食的状态。

- Docker 的相关命令可直接使用
- 若希望 Atomic 服务器能和私有源 pull、push 容器镜像，需要安装 docker-distribution
- 用于容器编排的 kubernetes 软件也被预装完成，只是相关服务默认不自动启

> 如若希望做 Docker 镜像的开发，则最好在标准的 RHEL7 上去做，这样才能安装所需的各种工具。
