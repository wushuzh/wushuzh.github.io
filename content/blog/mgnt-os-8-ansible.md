+++
date = "2017-09-21T21:07:09+08:00"
title = "用安赛波自动化管理"
showonlyimage = false
image = "/img/blog/mgnt-os-8-ansible/ansible-wave.png"
topImage = "/img/blog/mgnt-os-8-ansible/2voice_recognition_process_by_gleb.gif"
draft = false
weight = 69
+++

学着像 Ender 一样高效操控战队
<!--more-->

##

上篇 ["打造虚拟实验厂"]({{< relref "blog/mgnt-virenv-7-vagrant.md" >}}) 是借助 Vagrant 创建虚拟机。本篇则是借助 ansible 自动化的完成配置、部署。

提升效率最关键的办法就是将“一次投入，多次使用”

- 相比制造业，软件操作的标准化将开发、测试、生产环境无限趋向于一致
- 唯有抽象和封装才能实现更为复杂的操作
- 人有极限，会犯错，让机器去执行更靠谱

ansible 最大的特点大概就是它不需要 agent，非常容易上手。

{{< highlight console >}}
$ sudo pacman -S ansible

{{< /highlight >}}

参考文档

> - http://www.intigua.com/blog/puppet-vs.-chef-vs.-ansible-vs.-saltstack
> - http://blog.takipi.com/deployment-management-tools-chef-vs-puppet-vs-ansible-vs-saltstack-vs-fabric/


封面图片来自 [Voice recognition process exploration for AI product](https://dribbble.com/shots/3477540-Voice-recognition-process-exploration-for-AI-product) <a href="https://dribbble.com/glebich"><i class="fa fa-dribbble" aria-hidden="true"></i> Gleb Kuznetsov✈</a>
