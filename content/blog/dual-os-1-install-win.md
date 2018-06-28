+++
date = "2017-08-03T19:48:55+08:00"
title = "双系统 1: 微软视窗"
showonlyimage = false
image = "/img/blog/dual-os-1-install-win/mss_hero.png"
topImage = "/img/blog/dual-os-1-install-win/ms_surface.gif"
draft = false
weight = 61
+++

谁都不太可能绕得过 Windows ……
<!--more-->

## 硬件评估

我感觉自己工作用的笔记本快不行了，同时开个浏览器和 IDE ，风扇的动静像直升机起飞，键盘周围也是非常暖和，每天都像是在做免费的腕部热敷。

- ~~Intel i5 2.6 GHz 处理器 ( 2 核 4 线程 ) DDR3 4G 内存 ( HP 6470b Win7)~~
- Intel i5-7300U 2.6 GHz ( 2 核 4 线程 ) DDR4 8G 内存 (201805 联想 T470 Win10）

于是找来个老台式机分担一些工作负荷：准备装双系统，因为你总有一些时刻依赖 Windows 或 IE ，而 Linux 则用来应对偶尔 Lab 断网没服务器用的情况。

- ~~Intel i5 3.2 GHz 处理器 ( 2 核 4 线程 )~~
- ~~DDR3 8G 665 MHz 内存~~
- ~~Nvidia Quadro [NVS 290] 256MB显存~~
- Intel Xeon X5650 2.67 GHz
- DDR3 12G 内存
- NVIDIA GT218 [NVS 300] 512MB
- NVIDIA GT216GL [Quadro 400] 512MB

{{< highlight bat >}}
REM CMD 直接运行
msinfo32
dxdiag

REM 第三方工具 CPU-Z 或 GPU-Z 查看则需安装
{{< /highlight >}}

对于双系统，网上大部分建议都是先装 Windows，再装 Linux。而多数 PC 预装 Windows (据说少数还预装 Linux )。但我这台 PC 原装 Windows 已经没了，公司 IT 一般都是用克隆，不知道是不是绑定硬件的一种特定的批量 license 授权。

好在我自己之前买过一份正版 Win 8，先拿来用。下载 Win 8 安装介质做成一个可启动 U 盘——这年头光驱算古董。Windows 的安装没什么可说，一步步照提示确认就能安装好。现在 Windows 安装挺简洁，半小时绝对搞定了。

硬件大换血后 Windows 系统需要重新人工认证，这部分大概需要联系微软，回头走一边流程后再补上来。

## 系统更新

安装成功后，第一步就是联网，将系统补丁打到最新。

> “控制面板->程序-> Windows 更新 -> 全选并下载安装->重启”。

这个过程大概要重复几次，才能真正把系统更新完毕。要是你后面还装了 Office 、Visual Studio 记得回来再做几次系统补丁更新。

期间还按照[这篇文档](http://xyz.cinc.biz/2015/04/windows-w32tm.html)做了一下 NTP 时钟同步。（反正这台 PC 不能找 IT 加入域，没法用域时间同步服务器）


## 日常工具

Windows 上除了一些一次性安装的工具，其他常用软件的安装、升级、卸载应该用微软官方的包管理工具[Chocolatey](https://en.wikipedia.org/wiki/NuGet) ，安装很简单，到官网复制最新的安装命令，启动一个带管理员权限的命令行窗口，粘贴执行就可以了。

之后的软件安装升级都通过命令行，cinst 将待安装软件一个个搞定。

- 7zip
- listary
- Cmder (但中文内容显示的问题尚未解决)
- openssh
- Google Chrome
- Firefox
- vlc
- vagrant
- virtualbox
- vim
- notepadplusplus
- gimp
- greenshot

<br />

{{< figure src="/img/blog/dual-os-1-install-win/platinum.png" title="linux foundation platinum member" >}}

> 最近(2018-06)新闻提到腾讯成为了 Linux 基金会的铂金会员(最高等级)，然后去查看了一下当下 [15 个白金玩家](https://www.linuxfoundation.org/membership/)，这里面居然有微软……不过公平的说，微软这几年确实在开源世界屡放大招，代码编辑器 vscode 可用于全平台，开发出 [Windows Subsystem for Linux (WSL)](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux) ，包括最近收购 github，而其股价也从 2014 年初的 40 升至 2018年6月 100 美元每股)


## 准备 Linux 分区

情报是行动的前提，先查看分区表是老式 MSDOS 或新型 GPT 。运行命令行工具``` diskpart ```，执行``` list disk ``` 就可以确认。

查到的结果我的 PC 不是 GPT。之后退出命令行工具：

- 启动一个图形化分区工具 ``` diskmgmt.msc ```
- 右键选了除系统保留外的唯一的 C 盘，点击“压缩卷” (Shrink Volume)，得到一个未分配的分区
- 用一部分空间，新建一个分区用来存放数据，而剩下的部分会被用于 Linux 的安装

<br />

## 禁止 Win8 快启

Windows 8 引入了一个新功能，就是将其内核和设备驱动保存到磁盘上。这样下次开机就省去了和冷启动相关的各种初始化，能把启动速度提升 30 % ~ 70 %。但这个技术在双系统却会导致 Linux 启动后无法访问 Windows 的分区以及一些硬件的功能异常，所以还是禁掉吧。步骤见[How to Turn "Fast Startup" On or Off for a Hybrid Boot in Windows 8 and 8.1](https://www.eightforums.com/tutorials/6320-fast-startup-turn-off-windows-8-a.html)

## 准备安装介质

依然是准备一个可启动的 U 盘，写入 Linux 的安装介质。步骤按照[USB flash installation media](https://wiki.archlinux.org/index.php/USB_flash_installation_media)

{{< figure src="/img/blog/dual-os-1-install-win/Cold_HybridBoot.png" title="Cold vs HybridBoot" >}}

### 参考文档

> - One Transistor (2015-12-03) [Dual booting Windows and Ubuntu [MBR]](
https://onetransistor.blogspot.fr/2015/03/dual-boot-windows-and-ubuntu.html)
> - jdhao (2017-12-29) [cmder --- the Ultimate Command Line Tool for Windows](https://jdhao.github.io/2017/12/29/cmder-usage/)
> - jdhao (2017-03-01) [An Introduction to Listary: A Blazing Fast Search Tool](https://jdhao.github.io/2017/03/01/Listary_intro/)

封面图片来自 [Microsoft Surface Studio](https://dribbble.com/shots/3076441-Microsoft-Surface-Studio) <a href="https://dribbble.com/ercannailoglu"><i class="fa fa-dribbble" aria-hidden="true"></i> Ercan Nailoğlu</a>  
