+++
date = "2020-01-20T10:07:18+08:00"
title = "ARTS 2020w03"
image = "/img/blog/arts-84-per-week/"
draft = true
weight = 2003
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

遇到一个问题：在 archlinux 上通过 vagrant 创建的 kvm 虚拟机不能访问互联网。

问题分宿主机和虚拟机两个界面解决：

1. 对于宿主机 archlinux 上的域名解析排查，在上一篇已经涉及。

2. 对于基于 kvm 的虚拟机，vagrant 默认创建一个名为 vagrant-libvirt 的 192.168.121.0/24 网络，并设置了 NAT。 


{{< highlight txt >}}

$ virsh net-dumpxml vagrant-libvirt
<network connections='1' ipv6='yes'>
  <name>vagrant-libvirt</name>
  <uuid>25e0aa29-6f96-471c-bf27-9bf270a47705</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr2' stp='on' delay='0'/>
  <mac address='52:54:00:9c:f6:f9'/>
  <ip address='192.168.121.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.121.1' end='192.168.121.254'/>
    </dhcp>
  </ip>
</network>
{{< /highlight >}}

## Share


{{< highlight txt >}}
$ sudo groupadd docker
$ sudo gpasswd -a myuser docker
Adding user myuser to group docker
$ id myuser
uid=1000(myuser) gid=985(users) groups=985(users),998(wheel),1000(docker)

minikube start --insecure-registry=registry.kube-system.svc.cluster.local:80 --image-mirror-country=cn --registry-mirror=https://registry.docker-cn.com --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
minikube addons enable registry
✅  registry was successfully enabled
$ kubectl get svc --namespace=kube-system
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   3h1m
registry   ClusterIP   10.96.193.154   <none>        80/TCP                   3s


minikube start --image-mirror-country=cn --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers

{{< /highlight >}}

封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>