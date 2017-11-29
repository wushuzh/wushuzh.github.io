+++
date = "2017-11-28T20:44:11+08:00"
title = "Nmap 扫描"
showonlyimage = false
image = "/img/blog/use-nmap-to-scan/site_zero.png"
draft = false
weight = 110
+++

成片扫描网段和端口/服务的快捷方法
<!--more-->

### Nmap 工具

Nmap 用于发现网络中的主机和运行的服务，绘制网络地图。其工作原理就是发送定制包，然后分析回包。主要功能包括

- 发现主机/端口和服务
- 判定操作系统
- 和一些扩展脚本实现复杂检测，乃至漏洞检测

典型的应用场景分为 white-hat 和 black-hat

- 安全方面的审计，资产盘点，发现网络中新可疑节点
- 网络攻击窥探

{{< highlight console >}}
$ pacman -Ss nmap
extra/nmap 7.60-1
    Utility for network discovery and security auditing
community/vulscan 2.0-2
    A module which enhances nmap to a vulnerability scanner

nmap -T5 -sP 135.252.165.2-255
nmap -sS -p 22 135.252.165.2-255
{{< /highlight >}}


参考文档

> - [](https://hackertarget.com/nmap-tutorial/)
> - [](https://youtu.be/3Ab1gw8vQjg)

封面图片来自 [Site Zero ///](https://dribbble.com/shots/3751115-Site-Zero) <a href="https://dribbble.com/mjmurdock"><i class="fa fa-dribbble" aria-hidden="true"></i> mike murdock</a>
