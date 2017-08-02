+++
date = "2017-08-01T19:48:55+08:00"
title = "双系统 1: 微软视窗"
showonlyimage = false
image = "/img/blog/dual-os-1-install-win/windows_ui.png"
draft = false
weight = 61
+++

谁都不太可能绕得过 Windows ……
<!--more-->

我因为自己的笔记本性能太差，Intel i5 2.6 GHz 处理器(4核)，4 G 内存：比如同时开浏览器和 IDE ，风扇的动静像个直升机，然后一会儿键盘周围就热的像火焰山。不得不找个台式机来分担一些工作负荷。安装个双系统，是因为有公司内使用总有一些软件依赖 Windows 及其浏览器，再装个 Linux 来应对一旦 Lab 断网没服务器使用的情况。

对于双系统，网上大部分建议都是先装 Windows，再装 Linux。多数 PC 预装 Windows。但我这台 PC 原装 Windows 已经没有了，这种自己的 PC 公司 IT 肯定也不负责。好在我自己之前买过一份正版 Windows 8，先拿来用。之前早就把 Windows 8 的安装介质写入了 U 盘——这个年头，很多机器可能都没有光驱，还是用 U 盘靠谱。

Windows 的安装没什么可说，一步步照提示确认就能安装好。启动后第一步就是联网将系统补丁打到最新。“控制面板->程序-> Windows 更新 -> 全选并下载安装->重启”。这个过程大概要重复几次，才能真正把系统更新完毕。期间还按照[这篇文档](http://xyz.cinc.biz/2015/04/windows-w32tm.html)做了一下 NTP 时钟同步。（反正这台 PC 不能找 IT 加入域，没法用域时间同步服务器）

## 安装工具

Windows 上除了一些一次性安装的工具，其他常用的软件应该用脚本来搞定。先安装 [Chocolatey](https://en.wikipedia.org/wiki/NuGet)，到其官网上直接找到命令，启动一个有管理员权限的命令行窗口，粘贴执行就可以了。

之后再需要任何软件优先通过 choco 安装或升级，效率能提升不少。

## 准备 Linux 分区



封面图片来自 [Windows UI Concept](https://dribbble.com/shots/576250-Windows-UI-Concept) <a href="https://dribbble.com/Phyek"><i class="fa fa-dribbble" aria-hidden="true"></i> Phyek</a>  


https://support.hp.com/us-en/drivers/selfservice/hp-compaq-8180-elite-convertible-microtower-pc/4143434

drop UEFI support from GPT disk
fall back on BIOS mode from MBR/msdos disk only
  https://wiki.archlinux.org/index.php/Dual_boot_with_Windows

arch linux liveccd boot

parted /dev/sda

(parted) mkpart primary ext4 1MiB 20GiB
(parted) set 1 boot on
(parted) mkpart primary ext4 20GiB 100GiB

windows 8.1 usb boot

all 200 GB to windows, windows installer will create a 300 M reserve disk and a 197 GB partition

30 min finish Install and then upgrade HP PC BIOS


https://wiki.archlinux.org/index.php/GNU_Parted#BIOS.2FMBR_examples

fresh start

https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#Using_Windows_boot_loader

https://www.iceflatline.com/2009/09/how-to-dual-boot-windows-7-and-linux-using-bcdedit/

https://medium.com/@ashwinkailas/arch-linux-lvm-dual-boot-with-windows-tutorial-e327ea8f3140

UEFI/BIOS overview  https://support.hp.com/hk-en/document/c03801890#AbT3

https://technet.microsoft.com/en-us/library/hh825112.aspx
https://technet.microsoft.com/en-us/library/dn336946.aspx


https://www.howtogeek.com/193669/whats-the-difference-between-gpt-and-mbr-when-partitioning-a-drive/

https://lampjs.wordpress.com/2017/01/19/easy-installing-arch-linux-dual-boot-with-windows-uefi-or-mbr-for-beginners/

https://onetransistor.blogspot.fr/2015/03/dual-boot-windows-and-ubuntu.html
