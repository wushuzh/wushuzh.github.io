+++
date = "2017-09-01T22:01:43+08:00"
title = "软件定义热点"
showonlyimage = false
image = "/img/blog/dual-os-5-wifi-ap/freewifi.png"
draft = false
weight = 66
+++

利用无线网卡共享计算机的网络访问
<!--more-->

## 无线配置

安装 ArchLinux 系统时，建议一定是通过可靠的有线网络。但如果你使用的是笔记本，还是 wifi 更适合日常使用，因此学会确认驱动、配置无线就成了一个必要技能。尤其你还可以通过无线网卡让你的手机接入计算机上的有线网络——于是你又省下不少宝贵的 4G 流量——知识就是金钱。

### 网卡驱动

从光盘、软盘、网上下载、安装各种声卡、显卡驱动，大概是上个世纪每个自己攒过电脑，重装过 Windows 的同学都无法忘却的记忆。貌似从 Win7 开始，用户自行安装驱动成了一个很罕见的操作。

而另一方面，Linux 的硬核用户通常希望对自己的硬件拥有的更大程度的知情权和控制权，这使得自行安装、配置外设变得十分必要。

首先，Linux 内核仅在启动时，由 udev 为检测到的设备载入相应模块(驱动)。udev 面对成千上万种或来自正规大厂，或来自山寨创客的硬件无疑也可能出错，另外，linux-firmware 中很可能未能包含一些特殊硬件另外需要的固件。这也是为什么需要确认当下的驱动状态。

依据你的无线网卡插糟，选择执行 ```lspci -k``` 或 ```lsusb -v``` 确认设备当前的驱动。例如

{{< highlight console >}}
$ lsusb -t
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/8p, 480M
        |__ Port 6: Dev 3, If 0, Class=Vendor Specific Class, Driver=mt7601u, 480M
        |__ Port 7: Dev 4, If 0, Class=Human Interface Device, Driver=usbhid, 12M
        |__ Port 7: Dev 4, If 1, Class=Human Interface Device, Driver=usbhid, 12M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M
        |__ Port 2: Dev 3, If 0, Class=Audio, Driver=snd-usb-audio, 12M
        |__ Port 2: Dev 3, If 1, Class=Audio, Driver=snd-usb-audio, 12M
        |__ Port 2: Dev 3, If 2, Class=Audio, Driver=snd-usb-audio, 12M
        |__ Port 2: Dev 3, If 3, Class=Human Interface Device, Driver=usbhid, 12M


$ dmesg | grep usbcore
[    0.878139] usbcore: registered new interface driver usbfs
[    0.878149] usbcore: registered new interface driver hub
[    0.878176] usbcore: registered new device driver usb
[    1.743256] usbcore: registered new interface driver usbhid
[   10.266685] usbcore: registered new interface driver snd-usb-audio
[   11.906657] usbcore: registered new interface driver mt7601u

$ dmesg |grep mt7601u
[   10.967420] mt7601u 2-1.6:1.0: ASIC revision: 76010001 MAC revision: 76010500
[   11.005029] mt7601u 2-1.6:1.0: Warning: unsupported EEPROM version 0d
[   11.005033] mt7601u 2-1.6:1.0: EEPROM ver:0d fae:00
[   11.005651] mt7601u 2-1.6:1.0: EEPROM country region 01 (channels 1-13)
[   11.906657] usbcore: registered new interface driver mt7601u
[   12.177970] mt7601u 2-1.6:1.0 wlp0s29u1u6: renamed from wlan0

{{< /highlight >}}

### 参考文档

> -

封面图片来自 [Free Wifi](https://dribbble.com/shots/842806-Free-Wifi) <a href="https://dribbble.com/Tuttle"><i class="fa fa-dribbble" aria-hidden="true"></i> Tuttle</a>  
