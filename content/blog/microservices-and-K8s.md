+++
date = "2017-10-24T22:20:04+08:00"
title = "K8s容器管理"
showonlyimage = false
image = "/img/blog/microservices-and-K8s/captain_dock.png"
topImage = "/img/blog/microservices-and-K8s/captain_dock.gif"
draft = false
weight = 110
+++

啊，船长，啊，我的船长……
<!--more-->

想当年，Google 选择发表大数据论文而不是开源相关技术，使得山寨版 Hadoop 迅速出现，大行其道，自己反而被拍死在沙滩上。

对于这回的容器浪潮，Google 显然是吸取了教训，直接放出了 Kubernete ，在最终容器管理的争夺战硝烟散去后，我们看到老船长终是屹立不到。(~~Swarm~~, ~~Mososphere~~)

本地搭建，并最快上手的路经就是运行一遍其官方发布的 [hello-minikube](https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/) 

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
