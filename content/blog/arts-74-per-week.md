+++
date = "2019-11-18T16:39:48+08:00"
title = "ARTS 2019w47"
image = "/img/blog/arts-74-per-week/"
draft = true
weight = 1947
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

## Share

今年 6 月，树莓派 4 发布。但我手边只有一个 Raspberry PI 3，最近试着在上面运行 archlinux 。系统准备步骤如下：

1. 到 archlinuxarm [下载页面](https://archlinuxarm.org/about/downloads)，找到对应版本下载镜像，比如树莓派 3 应该对应下载 [ARMv8 Raspberry Pi 3](http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-3-latest.tar.gz)
2. 准备一个读卡器和 microSD卡，按照 [armv8 raspberry pi 3](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3) 的安装 tab 页面的提示一步步将 1 下载的镜像解压到 microSD 卡中
3. 将 microSD 卡插入树莓派，准备好外设，建议使用一个支持 HDMI 接口的显示器和 HDMI 连接线，一个USB蓝牙键鼠，一个能提供 5v 的 microUSB 接口的电源线；
4. 接通电源，启动，默认机器名为 alarm，普通用户 alarm，超级用户 root，默认密码同用户名；

值得注意的是，archlinux arm 是一个独立项目和社区，archlinux 官方对前者并不做官方支持。两个系统的使用方法当然并无差别，所以用户可以参看 archlinux 上完善的 wiki。

树莓派 3 已经自带 wifi，连接后最重要的一件事就是连网，更新系统，下载软件。

以 [WPA_supplicant](https://wiki.archlinux.org/index.php/WPA_supplicant) 的首次启动，交互式命令行配置为例，

0. 切换到 root 用户；
1. 创建配置文件，然后启动程序 `wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf`；
2. 运行交互式命令行 `wpa_cli`，扫描可用热点，设定连接用户名，密码，保存，连接；
3. 运行 `dhcpcd wlan0` 获取 IP 地址；
4. 编辑 `/etc/pacman.d/mirrorlist` 开启临近的的镜像地址，比如新加坡，运行系统更新，测试网络；

确认能上网成功后，就可以将上述过程配置为自动启动：[启动 dhcpcd](https://wiki.archlinux.org/index.php/Dhcpcd#Running) 并[创建 hook 连接](https://wiki.archlinux.org/index.php/Dhcpcd#10-wpa_supplicant)

> 转换到上述过程中，发现 pacman 联网不成功，`error: failed retrieving file 'xxx' from sg.mirror.archlinuxarm.org : Resolving timed out after 10000 milliseconds`。对应的 journalctl 出现 `systemd-resolved[418]: DNSSEC validation failed for question sg.mirror.archlinuxarm.org IN AAAA: failed-auxiliary` 报错信息，更新配置文件 `/etc/systemd/resolved.conf` 中 `DNSSEC=no` 后解决。

封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>