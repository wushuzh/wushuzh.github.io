+++
date = "2017-07-06T09:28:56+08:00"
title = "搭建容器的专属系统"
showonlyimage = false
image = "/img/blog/setup-a-container-host-env/redhat-docker.png"
draft = false
weight = 51
tags = ["virtulization", "redhat"]
+++

Docker 简介和 Atomic 服务器的搭建
<!--more-->

## Docker 起源

以往说起虚拟化，用的软件普遍是 VirtualBox，VMWare系，KVM，Oracle VM 。最终得到也都是 “ 完全虚拟化的 VM ”——即对整个计算机系统的仿真 ( 如利用开源 QEMU 实现的 Hypervisor )

下面的 “ 操作系统级虚拟化 ” 则是虚拟化科技树上的另一股分支：在一个内核上，分隔出多个用户空间，每个实例称为“容器” 或 “jails”。其来源是 Unix 的 chroot 操作。随着隔离安全、资源预留等 Linux 新特性的不断发展，这个新的技术解决方案开始进入我们的视野：集装箱改变世界。

## Docker 背后技术

![Key tech of container](/img/blog/setup-a-container-host-env/lxc_architecture.png)

- Namespaces
  * Mount NS 使容器可以拥有各自独立的文件系统
  * UTS NS 使容器拥有独立的主机或 NIS 域名
  * IPS NS 使容器内的进程间通信(共享内存、信号量)成为可能
  * PID NS 使容器有独立的 PID1 以及 /proc 从宿主机会看到不同的 PID
  * Nwk NS 使容器有各自独立的网络地址、防火墙及路由表
  * User NS 和容器用户权限相关，RHEL7 暂时未支持
- cgroups 用于对 CPU，内存，网络等[资源管控](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Resource_Management_Guide/index.html)
- SELinux 可从宿主机对容器应用各种 [SELinux 的规则和标签](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SELinux_Users_and_Administrators_Guide/chap-Security-Enhanced_Linux-sVirt.html)，比如 virtd_lxc_t

## Docker 镜像存储

Docker 将目标应用和所依赖的运行环境合成于容器中，独立于宿主 OS 。
Docker 的镜像格式底层是 device mapper thin provisioning 技术( 实现了copy-on-write 的 LVM 快照技术的演进型 )

![Docker Image](/img/blog/setup-a-container-host-env/docker_structure.png)

- Container App 运行所处的层级，基于 image 而启动，最上为可写层，可提交为新 image layer
- Image 则是不可更改的容器静态快照，每个 Image 都依赖于其父 Image，追溯到根源是 Platform Image (包含最最基础的指令、环境、软件包)

将隔离的应用，实现标准化的构建和部署。
无论对于软件开发，还是基础设施，都可以创建标准可重复的流程。

## 和虚拟机比较

- KVM 全面仿真硬件，独立 OS 内核，有更好的隔离和安全性，甚至仿真不同架构，安装 Windows/Mac 虚拟机
- 容器 则是以应用为中心，所以共享宿主上的内核及关键程序，消耗更好资源，是超轻量级的虚拟化技术

## Atomic 服务器

Redhat 围绕 LDK 技术栈、基于基础设施应固化的理念，将操作系统进行了重新设计和瘦身，成为一个容器 OS 诞生出来，名为 Atomic Host 。

和以往操作系统的不同有：

- 使用 rpm-ostree 而非 Yum 做操作系统中的包管理
- 仅两个目录 /etc/ 和 /var/ 可写，升级时采用新老引导系统并存保留的方式
- 存储由守护进程 docker-storage-setup 负责，利用 LVM 创建 root 和 docker-pool 两个逻辑卷
- 特有的为容器 OS 而设定的安全特性
- 能和其他来自 Redhat Openshift 的管理工具全家桶配合使用
- https://access.redhat.com/articles/2772861

<br />

### 动手环节
<img alt="XKCD #1764" src="/img/blog/setup-a-container-host-env/xkcd-1764.png" class="img-responsive">

对 Atomic 新系统的尝鲜，最简便的是通过虚拟机：

- RedHat的订阅用户，亦或 CentOS、Fedora 各自的 Atomic SIG 都提供可用于云环境的 QCOW2 的镜像[下载](http://www.projectatomic.io/download/)。可在 OpenStack，oVirt，virt-manager 下使用。
- Vagrant 格式的镜像可以在 VirtualBox 及 libvirt/KVM 模式下使用；
- AMI 格式的镜像可用于 EC2 环境；

按照 Matthew M. (2014-10-21) [Getting started with cloud-init](http://www.projectatomic.io/blog/2014/10/getting-started-with-cloud-init/) 介绍的步骤，我启动了一台容器宿主虚拟机。后来发现应该用 Atomic 官网上的文档  [qucikstart](https://www.projectatomic.io/docs/quickstart/) ，再后来发现应该用 [RedHat Atomic](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html-single/installation_and_configuration_guide/) 的官网文档 —— VM Installation 一节。

主要步骤如下：

1. 下载解压 QCOW2 的镜像到本地，用 qemu-img 为新 VM 准备相应的 QCOW2 文件
2. 为 clould-init 准备 ISO ，包含 metadata、userdata 两个含如下元数据的 YAML
   - 如虚拟机实例名，主机名
   - 如用于登录的 SSH 公钥，以及是否允许密码登录
3. 之后有两种方式创建虚拟机实例:
   - 使用图形化工具 virt-manager 创建虚拟机，过程和一般虚拟机无异，除了触发安装前要添加 cloud-init ISO 到 CDROM。
   - 使用命令行工具 virt-install import 通过镜像和 ci ISO 创建虚拟机，Giovanni Torres (2016-05-11) [Create a Linux Lab on KVM Using Cloud Images](http://giovannitorres.me/create-a-linux-lab-on-kvm-using-cloud-images.html) 中提供了一个很完备的 shell 脚本，并在 [GitHub 上维护](https://github.com/giovtorres/kvm-install-vm)
5. 可选步骤：为 VM 添加存储，登录 VM 做系统更新，开始使用 （更详尽的试用报告见本系列下篇）

此时，你应该至少创建了两台虚拟机，一台 Atomic，一台 CentOS 7/RHEL 7。

缩略语解释

LDK: Linux Docker Kubernetes
SIG：Special Interest Group
