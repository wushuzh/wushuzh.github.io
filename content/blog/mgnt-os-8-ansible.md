+++
date = "2017-09-21T21:07:09+08:00"
title = "用安赛波自动化管理"
showonlyimage = false
image = "/img/blog/mgnt-os-8-ansible/ansible-wave.png"
topImage = "/img/blog/mgnt-os-8-ansible/2voice_recognition_process_by_gleb.gif"
draft = true
weight = 69
tags = ["ansible"]
+++

学着像 Ender 一样高效操控战队
<!--more-->

## 缘起

上篇 ["打造虚拟实验厂"]({{< ref "/blog/mgnt-virenv-7-vagrant.md" >}}) 是借助 Vagrant 创建虚拟机。本篇则是借助 ansible 自动化的完成配置、部署。

提升效率最关键的办法就是将“一次投入，多次使用”

- 向制造业学习，通过标准化操作倒逼软件开发、测试、生产环境无限趋向于一致
- 模块化抽象和封装是实现更为复杂的自动操作的基石
- 人有极限，会犯错，让机器去执行重复任务才更靠谱

ansible 最大的特点是它不需要 agent，文档健全，容易上手。

{{< highlight console >}}
$ sudo pacman -S ansible

{{< /highlight >}}

## 迷你环境

使用 vagrant 创建一个小集群

## 仿真环境

在 Openstack 环境下，ansible 也可以管理 instance(s)。

在 ansible 管理节点上:

1. 准备好用于创建实例的 pem 文件
2. 将所有要管理的 VM 的 IP、主机名加入到 /etc/hosts 文件中，并据此组织 inventory 文件。下面两种方案都可以屏蔽首次连接时 fingerprint 的问题。
  - ansible conf 中```host_key_checking = False``` 或
  - ```ssh-keyscan vm-list >> ~/.ssh/know_hosts```
3. 确认实例的 Security Groups 的相关 Rules 配置正确，即 ssh 链接被允许，我通过 RHEL Generic Cloud image 创建的 VM ，所以使用 cloud-user 连接
4. 创建本地 ansible.cfg 设定指向第 1, 2 步准备的 private_key_file 和 inventory 文件
5. 创建 playbook 运行 ping 模块，记得要设定 remote_user = cloud-user

## 模块

ping
timezone # setup ntp
filesystem
mount
yum_repository
copy
blockinfile


参考文档

> - Justin Weissig Ansible series
  - [1 Intro  Ansible](https://sysadmincasts.com/episodes/43-19-minutes-with-ansible-part-1-4)
  - [2 Learning with Vagrant ](https://sysadmincasts.com/episodes/45-learning-ansible-with-vagrant-part-2-4)
  - [3 Conf Mgmt](https://sysadmincasts.com/episodes/46-configuration-management-with-ansible-part-3-4)
  - [4 Zero-downtime Deployment](https://sysadmincasts.com/episodes/47-zero-downtime-deployments-with-ansible-part-4-4)
> - http://www.intigua.com/blog/puppet-vs.-chef-vs.-ansible-vs.-saltstack
> - http://blog.takipi.com/deployment-management-tools-chef-vs-puppet-vs-ansible-vs-saltstack-vs-fabric/

封面图片来自 [Voice recognition process exploration for AI product](https://dribbble.com/shots/3477540-Voice-recognition-process-exploration-for-AI-product) <a href="https://dribbble.com/glebich"><i class="fa fa-dribbble" aria-hidden="true"></i> Gleb Kuznetsov✈</a>
