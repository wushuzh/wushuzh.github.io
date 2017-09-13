+++
date = "2017-09-01T22:01:43+08:00"
title = "软件定义热点 1"
showonlyimage = false
image = "/img/blog/dual-os-5-wifi-ap/freewifi.png"
draft = false
weight = 66
+++

利用无线网卡创建热点，共享网络访问
<!--more-->

## 无线 AP

安装 ArchLinux 系统时，用户手册会建议使用可靠的**有线**网络连接。

但一个装好的日用笔记本，wifi 显得更为方便。因此配置无线设备的驱动就成了基本技能。尤其我们还可以借助通过无线网卡建立热点，让手机共享计算机的网络——这样有又能省下不少移动流量。

这篇就是尝试自行为网卡安装配置的流水帐系列的第一篇。记录了我对 AUR 安装包的一通折腾，最终还没有成功的操作过程。

我手上有一个小米随身 wifi，Win8 下安装小米自己的驱动程序，确认可以当作 AP 使用。

> 说起来，需要用户自行安装、配置声/显卡等设备的驱动，大概是上世纪攒过电脑，装过“温八”同学的专属记忆了。而好像从 Win7 开始，不知是设备接口的标准逐渐统一，还是操作系统中预置了不少大厂驱动的缘故，自行安装成了非常罕见的操作。

小米需要安装 Windows 下的私有驱动当然有它的理由，但也预示着在 Linux 下的过程不会特别顺利。Linux 上各种配置的攻略文档可能会分散在github、论坛，个人博客上，不可避免地需要自己到处查找，但好处是你可以从中学到不少新东西，同时也许获得了对硬件最大程度的知情权和控制权。

### 查找设备

在 Linux ，一般是在 kernel 启动时，由 udev 为检测到的设备载入相应模块(驱动)。udev 面对成千上万种或来自正规大厂，或来自山寨创客的硬件无疑也可能出错，另外，linux-firmware 中很可能未能包含一些特殊硬件额外需要的固件。这也是为什么你需要首先确认当下的驱动状态是否正确。

依据你的无线网卡插入方式，选择执行 ```lspci -k``` 或 ```lsusb``` 确认设备当前的驱动。例如我用的是小米随身wifi，插在 usb 口上。

{{< highlight console >}}
$ lsusb -t
/:Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
  |_Port 1: Dev 2, If 0, Class=Hub, Driver=hub/8p, 480M
    |_ Port 6: Dev 3, If 0, Class=Vendor Specific Class, Driver=mt7601u, 480M
    |_ Port 7: Dev 4, If 0, Class=Human Interface Device, Driver=usbhid, 12M
    |_ Port 7: Dev 4, If 1, Class=Human Interface Device, Driver=usbhid, 12M
/:Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
  |_Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M
    |_ Port 2: Dev 3, If 0, Class=Audio, Driver=snd-usb-audio, 12M
    |_ Port 2: Dev 3, If 1, Class=Audio, Driver=snd-usb-audio, 12M
    |_ Port 2: Dev 3, If 2, Class=Audio, Driver=snd-usb-audio, 12M
    |_ Port 2: Dev 3, If 3, Class=Human Interface Device, Driver=usbhid, 12M

$ lsusb
Bus 002 Device 004: ID 046d:c52e Logitech, Inc. MK260 Wireless Combo Receiver
Bus 002 Device 003: ID 2717:4106
Bus 002 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 003: ID 0b0e:0301 GN Netcom
Bus 001 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

{{< /highlight >}}

> 执行 ```lsusb -s busid:devid -v```可以显示单个设备更详细的各式参数。

网上通过 ID 2717：4106 查找设备，比如这里 [wikidevi MediaTek MT7601U](https://wikidevi.com/wiki/MediaTek_MT7601U) 提到了这个设备被用在了小米 wifi 上。

### 确认驱动

在 dmesg 中过滤 usbcore 关键字，找出被注册为网络接口的设备，也能找到 mt7601u 和 wlp0s29u1u6 就是这个迷你 wifi 设备。

{{< highlight console >}}
$ dmesg | grep usbcore
[  0.878139] usbcore: registered new interface driver usbfs
[  0.878149] usbcore: registered new interface driver hub
[  0.878176] usbcore: registered new device driver usb
[  1.743256] usbcore: registered new interface driver usbhid
[ 10.266685] usbcore: registered new interface driver snd-usb-audio
[ 11.906657] usbcore: registered new interface driver mt7601u

$ dmesg |grep mt7601u
[ 10.967420] mt7601u 2-1.6:1.0: ASIC revision: 76010001 MAC revision: 76010500
[ 11.005029] mt7601u 2-1.6:1.0: Warning: unsupported EEPROM version 0d
[ 11.005033] mt7601u 2-1.6:1.0: EEPROM ver:0d fae:00
[ 11.005651] mt7601u 2-1.6:1.0: EEPROM country region 01 (channels 1-13)
[ 11.906657] usbcore: registered new interface driver mt7601u
[ 12.177970] mt7601u 2-1.6:1.0 wlp0s29u1u6: renamed from wlan0

{{< /highlight >}}

执行```ip link```确实能看到网口，```sudo ip link set wlp0s29u1u6 up```，没报错。

但 dmesg 中 ```grep wlp0s29u1u6```有一堆 ```IPv6: ADDRCONF(NETDEV_UP): wlp0s29u1u6: link is not ready```，进而为了确认设备支持的接口模式，安装 iw 工具，显示当下不支持 AP 模式。

{{< highlight console >}}
$ iw list
Wiphy phy0
...
  Supported interface modes:
    * managed
    * monitor
  Band 1:
    Capabilities: 0x17e
...
  Supported commands:
    ...
    * start_ap
    ...
	Supported TX frame types:
    ...
    * AP: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
    * AP/VLAN: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
    ...
	Supported RX frame types:
    ...
    * AP: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
    * AP/VLAN: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
    ...
	software interface modes (can always be added):
    * monitor
    ...
	Device supports AP scan.
...
{{< /highlight >}}

## 安装驱动

首先到 Archlinux 官方和 AUR 上搜索，果然找到了一个来自 [nous](https://aur.archlinux.org/account/nous) 的 [mt7601u-ap-dkms ](https://aur.archlinux.org/packages/mt7601u-ap-dkms/) 版本 3.0.0.3-1 但从评论上看，貌似会装不成。

无论如何，先试试再说。
{{< highlight console >}}
$ git clone https://aur.archlinux.org/mt7601u-ap-dkms.git
$ cd mt7601u-ap-dkms
$ makepkg -si
==> Making package: mt7601u-ap-dkms 3.0.0.3-1 (some date)
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
  -> Downloading master.zip...
  -> Found mt7601u-ap.conf
  -> Found dkms.conf
==> Validating source files with sha256sums...
    master.zip ... FAILED
    mt7601u-ap.conf ... Passed
    dkms.conf ... Passed
==> ERROR: One or more files did not pass the validity check!
{{< /highlight >}}

先查看 PKGBUILD 内容，确认是现在从 github 下载的 zip 文件的 sha256sum 的值变化了导致的。

AUR 上 shyokou (貌似是位中国同学)的评论，也确认了 sha256sum 对 git 没意义。之后 pcjason 将一个补丁直接贴到了评论区，更新了 mt7601u 的 github 驱动下载地址，并采用了 rin (2016-03-16) [在树莓派2上使用MT7601芯片USB网卡搭建AP](https://lo-li.net/1290.html) 提到的 C 代码补丁。既然如此将补丁内容保存为 pcjason.patch 文件尝试一下。

{{< highlight console >}}
$ git apply pcjason.patch
error: corrupt patch at line 7
{{< /highlight >}}

又查了下报错 应该是粘贴到评论区时行首空格行尾换行字符丢失造成的。尝试更改后，--stat 可以，但 apply 仍然报错。

{{< highlight console >}}
$ git apply --stat pajason.patch
 PKGBUILD         |   19 ++++++++++---------
 mt7601u-ap.patch |   17 +++++++++++++++++
 2 files changed, 27 insertions(+), 9 deletions(-)
{{< /highlight >}}

最后实在没有耐心调整各种空格了，更改行数也不太多，手动修改搞定貌似更容易一些。而后为了编译又安装了 linux-headers ，涉及内核升级，还重启了一次系统，但最终安装时还是报错。报错结果贴在了原 AUR 评论区。

{{< highlight console >}}
(1/1) installing mt7601u-ap-dkms [----------------------] 100%
Creating symlink /var/lib/dkms/mt7601u-ap/3.0.0.4/source ->
/usr/src/mt7601u-ap-3.0.0.4

DKMS: add completed.

Kernel preparation unnecessary for this kernel. Skipping...

Building module:
cleaning build area...
make -j4 KERNELRELEASE=4.12.10-1-ARCH.....(bad exit status: 2)
Error! Bad return status for module build on kernel: 4.12.10-1-ARCH (x86_64)
Consult /var/lib/dkms/mt7601u-ap/3.0.0.4/build/make.log for more information.
>>> You might need to modprobe mt7601Uap manually.
>>> Also, you *must* change the default values (especially WPAPSK)
>>> in /etc/Wireless/RT2870AP/RT2870AP.dat
>>> Read the documentation in /usr/src/mt7601u-ap-3.0.0.3/doc.
:: Running post-transaction hooks...
(1/2) Install DKMS modules
==> dkms install mt7601u-ap/3.0.0.4 -k 4.12.10-1-ARCH
Error! Bad return status for module build on kernel: 4.12.10-1-ARCH (x86_64)
Consult /var/lib/dkms/mt7601u-ap/3.0.0.4/build/make.log for more information.
(2/2) Arming ConditionNeedsUpdate...

{{< /highlight >}}

## 小结

虽然安装失败，但仍然挺有意思。以后有时间继续填这个坑。

### 参考文档

> - Archlinux wiki [Wireless network configuration](https://wiki.archlinux.org/index.php/Wireless_network_configuration)  
> - Archlinux wiki [Software access point](https://wiki.archlinux.org/index.php/Software_access_point)
> - Wikipedia [diff utility](https://en.wikipedia.org/wiki/Diff_utility)
> - ariejan de vroom (2009-10-26）[How to create and apply a patch with Git](https://www.devroom.io/2009/10/26/how-to-create-and-apply-a-patch-with-git/)
> - nixCraft (2005-12-14) [Zipping and Unzipping Files under Linux](https://www.cyberciti.biz/tips/how-can-i-zipping-and-unzipping-files-under-linux.html)

封面图片来自 [Free Wifi](https://dribbble.com/shots/842806-Free-Wifi) <a href="https://dribbble.com/Tuttle"><i class="fa fa-dribbble" aria-hidden="true"></i> Tuttle</a>  
