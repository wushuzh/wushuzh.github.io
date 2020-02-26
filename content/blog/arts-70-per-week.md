+++
date = "2019-10-21T23:49:51+08:00"
title = "ARTS 2019w43"
image = "/img/blog/arts-70-per-week/"
draft = true
weight = 1943
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

PXE启动一大批服务器安装后，若想批量检查哪些服务器被成功引导安装，可以借助如下方法：


安装 NMAP (有 MSWindows 版，nmap 在 WSL1 下无法使用，但理论上 WSL2 可以使用)，并通过下面的[命令扫描](http://thoughtsbyclayg.blogspot.com/2008/06/use-nmap-to-scan-for-ssh-servers-on.html)

{{< highlight txt >}}
nmap -p 22 --open -sV 1.1.1.0/24

Starting Nmap 7.80 ( https://nmap.org ) ...
Nmap scan report for 1.1.1.4
Host is up (0.038s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.4 (protocol 2.0)

{{ next sshable ip report ... }}

Nmap scan report for 172.29.128.252
Host is up (0.0057s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)

Nmap scan report for 1.1.1.253
Host is up (0.0040s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.3 (protocol 2.0)

Nmap scan report for 1.1.1.254
Host is up (0.0076s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     HP Comware switch sshd 7.1.070 (protocol 2.0)
Service Info: OS: Comware; CPE: cpe:/o:hp:comware:7.1.070

{{< /highlight >}}

辅助通过 DHCP 服务器的登录方法，查找已分配 IP 地址





## Share

如果用 choco 安装 wsl-debian 即使成功也找不到常见应用程序的图标，只能命令行启动

初始化 Debian 设置

adduser foousr
usermod -aG sudo foousr

sudo apt update
sudo apt install git


封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>