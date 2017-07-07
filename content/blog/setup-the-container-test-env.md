+++
date = "2017-07-06T09:28:56+08:00"
title = "setup the container test env"
showonlyimage = true
image = "/img/blog/setup-the-container-test-env/xkcd_1764.png"
draft = true
weight = 0
+++

## 容器的关键技术

![Key tech of container](/img/blog/setup-the-container-test-env/lxc_architecture.png)

- Namespaces
  * Mount NS 使容器可以拥有各自独立的文件系统
  * UTS NS 使容器拥有独立的主机或 NIS 域名
  * IPS NS 使容器内的进程间通信(共享内存、信号量)成为可能
  * PID NS 使容器有独立的 PID1 以及 /proc 从宿主机会看到不同的 PID
  * Nwk NS 使容器有各自独立的网络地址、防火墙及路由表
  * User NS 和容器用户权限相关，RHEL7 暂时未支持
- cgroups 用于对 CPU，内存，网络等[资源管控](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Resource_Management_Guide/index.html)
- SELinux 可从宿主机对容器应用各种 [SELinux 的规则和标签](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SELinux_Users_and_Administrators_Guide/chap-Security-Enhanced_Linux-sVirt.html)，比如 virtd_lxc_t

## 容器的应用方式

容器是将目标应用所依赖的运行环境合成于容器中，独立于宿主 OS 。
Docker 的镜像格式底层是 device mapper thin provisioning 技术( 实现了copy-on-write 的 LVM 快照技术的演进型 )

![Docker Image](/img/blog/setup-the-container-test-env/docker_structure.png)

- Container App 运行所处的层级，基于 image 而启动，最上为可写层，可提交为新 image layer
- Image 不可更改的容器静态快照，每个 Image 都依赖于其父 Image，追溯到根源是 Platform Image (包含最最基础的指令、环境、软件包)

## 和虚拟机的比较

- KVM 有独立于宿主的内核，更好的隔离和安全性，你甚至可以安装 Windows 虚拟机
- 容器 目的是运行独立的应用，所以共享宿主上的内核及关键程序，消耗更好资源，是超轻量级的虚拟化技术

## ATOMIC HOST 简介

使用 rpm-ostree 而非 Yum
仅两个目录 /etc/ 和 /var/ 可写，升级时采用新老引导系统并存保留的方式
存储由守护进程 docker-storage-setup 负责，利用 LVM 创建 root 和 docker-pool 两个逻辑卷
