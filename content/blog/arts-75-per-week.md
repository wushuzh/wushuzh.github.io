+++
date = "2019-11-25T16:39:48+08:00"
title = "ARTS 2019w48"
image = "/img/blog/arts-75-per-week/"
draft = true
weight = 1948
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

在 archlinux 上配置 minikube ：之前虚拟层试过 virtualbox ，这回我尝试使用 kvm2

前提是先（配置）[启动 libvirtd 服务](https://wiki.archlinux.org/index.php/Libvirt#Configuration)，记得更改 `/etc/nsswitch.conf` 以便可以通过 guest 主机名直接访问 VM

Failed to get bootstrapper
❌  Error: [KVM_CONNECTION_ERROR] getting kubeadm bootstrapper: command runner: getting ssh client for bootstrapper: Error creating new ssh host from driver: Error getting ssh host name for driver: machine in unknown state: getting connection: getting domain: error connecting to libvirt socket.: virError(Code=89, Domain=47, Message='error from service: CheckAuthorization: Did not receive a reply. Possible causes include: the remote application did not send a reply, the message bus security policy blocked the reply, the reply timeout expired, or the network connection was broken.')

sudo pacman -S minikube

第一次运行程序会试图下载各种依赖:
- `docker-machine-driver-kvm2` 到 `~/.minikube/bin`
- `minikube-v1.6.0.iso` 到 `~/.minikube/cache/iso`
- kube 相关服务的容器镜像 到 `~/.minikube/cache/images/<official-mirror-registry>/`
- kubeadm 和 kubelet 二进制文件 到 `~/.minikube/cache/v1.17.0/`

下载时如需借助代理，参见 [HTTP Proxies](https://minikube.sigs.k8s.io/docs/reference/networking/proxy/) ，特别是如果无法从 k8s.gcr.io 下拉 kube 相关服务的容器镜像，可以考虑使用国内大厂（阿里杭州）的同步镜像

minikube start --image-mirror-country cn


## Share

安装 libreoffice-still

封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>