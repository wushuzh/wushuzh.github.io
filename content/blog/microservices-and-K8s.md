+++
date = "2017-10-24T22:20:04+08:00"
title = "K8s容器管理"
showonlyimage = false
image = "/img/blog/microservices-and-K8s/captain_dock.png"
topImage = "/img/blog/microservices-and-K8s/captain_dock.gif"
draft = false
weight = 110
tags = ["kubernetes", "minikube"]
+++

啊，船长，啊，我的船长……
<!--more-->

想当年，Google 选择发表大数据论文而不是开源相关技术，使得山寨版 Hadoop 迅速出现，大行其道，自己反而被拍死在沙滩上。

对于这回的容器浪潮，Google 显然是吸取了教训，直接放出了 Kubernete ，在最终容器管理的争夺战硝烟散去后，我们看到老船长终是屹立不到。(~~Swarm~~, ~~Mososphere~~)

本地搭建，并最快上手的路经就是运行一遍其官方发布的 [hello-minikube](https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/) 

## Archlinux 

### 准备

首先是通过 minikube 工具创建/模拟一个本地集群。

{{< highlight console >}}
$ cower -d docker-machine-kvm
# note host folder sharing is not supported in kvm yet

$ cower -d kubectl-bin
$ cower -d minikube
# install above aur pkg ...

$ export http_proxy=ip:port
$ export https_proxy=ip:port
$ minikube start 
    --docker-env HTTP_PROXY=ip:port \
    --docker-env HTTPS_PROXY=ip:port \
    --docker-env NO_PROXY=192.168.99.0/24 \
    --vm-driver=virtualbox

$ minikube ip
192.168.99.100
$ minikube status
minikube: Running
cluster: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100

$ export no_proxy=192.168.99.100

$ kubectl config use-context minikube
$ kubectl config current-context
minikube

$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    47m       v1.10.0

$ kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   


{{< /highlight >}}

### 打包

准备一个 node 的 helloworld 程序。然后借用 Minikube VM 打包 docker 镜像。这样随后部署的时候就不需要 pull 啦。

{{< highlight console >}}
$ mkdir hellonode
$ touch hellonode/server.js
$ touch hellonode/Dockerfile
# get the code done
$ eval $(minikube docker-env --no-proxy)
$ docker build -t hello-node:v1 --build-arg HTTPS_PROXY=ip:port .
$ docker images
{{< /highlight >}}

### 部署

{{< highlight console >}}
$ kubectl run hello-node --image=hello-node:v1 --port=8080
deployment "hello-node" created
$ kubectl get deployments
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           1m

$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-69b47b745c-t28v7   1/1       Running   0          1m

$ kubectl get events

{{< /highlight >}}

### 成为服务

{{< highlight console>}}
$ kubectl expose deployment hello-node --type=LoadBalancer
service "hello-node" exposed
[wushuzh@zion hellonode]$ kubectl get services
NAME         TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.0.0.162   <pending>     8080:31564/TCP   3s
kubernetes   ClusterIP      10.0.0.1     <none>        443/TCP          29m

$ minikube service hello-node
# automatically open browser to ip of minikube-vm

$ kubectl logs hello-node-69b47b745c-t28v7
Received request for URL: /
Received request for URL: /favicon.ico

{{< /highlight >}}

### 更新服务

{{< highlight console >}}
$ docker build -t hello-node:v2 .
$ kubectl set image deployment/hello-node hello-node=hello-node:v2
deployment "hello-node" image updated

$ minikube service hello-node
# again browser with same url http://192.168.199.100:31564/

{{< /highlight >}}

### 下架清除

{{< highlight console>}}
$ kubectl delete service hello-node
service "hello-node" deleted
$ kubectl delete deployment hello-node
deployment "hello-node" deleted

$ minikube stop
Stopping local Kubernetes cluster...
Machine stopped.

{{< /highlight >}}

### Helm

{{< highlight console>}}
$ cower -d kubernetes-helm
# makepkg and install
$ helm version


{{< /highlight >}}

<br />

## Windows 10

如果是 Windows 7 这样老系统，通常依赖 Virtualbox 作为 Hypervisor 来创建相应的虚拟机及网络。Docker 这边配合安装的工具是 [Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/)

而现在 Windows 10 中微软内置了 Hyper-V ，Docker 为此发布了新的工具 [Docker For Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) ,其中 Edge 版本内置了 Kubernetes 的支持，省去了自行下载安装 kubectl 和 minikube 的操作，也很值得尝试。

上述两种方式我自己或是身边同事都有成功的经验。但我自己在 Windows 10 上的最终的安装方式却有点像上述两种的混合体——没有使用 Docker Toolbox 或是 Docker For Windows Edge 版本， 而是用 choco 直接安装 docker、kubernetes-cli 和 minikube 。

先是试图打开 Windows 10 的 HyperV 功能，用 minikube 创建基于 HyperV 的 VM，但当前版本貌似仍不稳定——首先是网上主流教程对于建立 HyperV Switch 分为两种方式: 

- 直接创建一个附着于网卡的 External 类型的 vSwitch
- 或者创建一个 Internal 类型的 vSwitch 然后通过 [NAT 方式](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network)和已有连接共享互联网

后来有人提到 Windows 10 的 1703 版本建议都使用默认 vSwitch —— 自带 NAT 功能。但使用后又遇到启动虚拟机拿不到 ipv4 地址的情况——需要通过手动关闭 vSwitch 的 ipv6 才能一定程度上解决这个问题，但得到的 ipv4 地址每次并不固定，而且启动后期在对 k8s.gcr.io 相应容器镜像的下载又总不成功。

最终我关闭了 HyperV 后转而使用 Virtualbox ，清除之前 .kube 和 .minikube 下的配置文件，设置一下 VBox 的代理，然后重新启动 minikube 虚拟机顺利创建配置，各种镜像下载也很顺利。

打开一个有管理员权限的 Powershell

{{< highlight console >}}

REM netsh winhttp set proxy "proxyip:port"
REM netsh winhttp show proxy

REM New-VMSwitch -SwitchName "minikube" -SwitchType Internal
REM Get-NetAdapter
REM New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex ??
REM New-NetNat -Name miniNAT -InternalIPInterfaceAddressPrefix 192.168.0.0/24
REM minikube start -v 9999 --vm-driver=hyperv --hyperv-virtual-switch="minikube" --docker-env http_proxy=proxyip:proxy --docker-env https_proxy=proxyip:port --docker-env no_proxy=192.168.0.0/24
REM 
REM minikube start -v 9999 --vm-driver="hyperv" --hyperv-virtual-switch="Default Switch" --alsologtostderr --docker-env http_proxy=proxyip:port --docker-env https_proxy=proxyip:port

minikube start -v 9999 --alsologtostderr --docker-env http_proxy=proxyip:port --docker-env https_proxy=proxyip:port --docker-env no_proxy=192.168.0.0/24
{{< /highlight >}}

参考文档

> - Tom Krazit (2017-10-17) [Docker acknowledges reality, adds support for Kubernetes to its container technology](https://www.geekwire.com/2017/docker-acknowledges-reality-adds-support-kubernetes-container-technology/)
> - Wikipedia [Kubernetes](https://en.wikipedia.org/wiki/Kubernetes)
> - Udacity ud615 [Scalable Microservices with Kubernetes](https://classroom.udacity.com/courses/ud615)
> - Kelsey Hightower Pycon 2017 [Kubernetes for Pythonistas](https://youtu.be/u_iAXzy3xBA)
> - [The Twelve-Factors App](https://12factor.net/)
> - 蝈蝈俊 (2016-07-25) [Conway's law(康威定律)](http://www.cnblogs.com/ghj1976/p/5703462.html)
> - Wikipedia JSON Web Token [JWT](https://en.wikipedia.org/wiki/JSON_Web_Token) and [Introduction to JSON Web Tokens](https://jwt.io/introduction/)
> - https://en.wikipedia.org/wiki/Transport_Layer_Security

封面图片来自 [Captain Animation](https://dribbble.com/shots/2287360-Captain-Animation)
 <a href="https://dribbble.com/kunchevsky"><i class="fa fa-dribbble" aria-hidden="true"></i> Zabombey</a>  
