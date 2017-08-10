+++
date = "2017-08-10T18:48:26+08:00"
title = "双系统 2: Linux"
showonlyimage = false
image = "/img/blog/dual-os-2-install-linux/linux-m.png"
draft = false
weight = 63
+++

选择哪个 Linux 版本？是个问题
<!--more-->

## Linux 版本

{{< figure src="/img/blog/dual-os-2-install-linux/linux-flavour.jpg" title="Decision Tree" >}}

上面的决策树送给犹豫不决的你。我选 Arch Linux

## 制作 U 盘
[官网](https://www.archlinux.org/download/)下载安装介质 ：我下载的是 2017.08.01 的版本，内核 4.12.3，ISO 大小 516 MB。
等待下载的同时，开个浏览器阅读 [USB flash installation media](https://wiki.archlinux.org/index.php/USB_flash_installation_media#In_Windows), 随便挑一种方法将 ISO 写入 U 盘。

{{< highlight console >}}
choco install dd -y

wmic diskdrive list brief
  Caption           DeviceID            Model  Partitions  Size
  Hitachi HDS72xxx  \\.\PHYSICALDRIVE0  ...    3           nnnnn
  Kingston Datayyy  \\.\PHYSICALDRIVE1  ...    1           mmmmm

dd if=archlinux-2017.08.01-x86_64.iso of=\\.\PHYSICALDRIVE1 bs=4M
  rawwrite dd for windows version 0.5
  ......
  129+0 records in
  129+0 records out

{{< /highlight >}}

### 参考文档

> - One Transistor (2015-12-03) [Dual booting Windows and Ubuntu [MBR]](
https://onetransistor.blogspot.fr/2015/03/dual-boot-windows-and-ubuntu.html)

封面图片来自 [Introduction to Linux](https://dribbble.com/shots/1862256-Introduction-to-Linux) <a href="https://dribbble.com/kabojanowska"><i class="fa fa-dribbble" aria-hidden="true"></i> Kasia</a>  
