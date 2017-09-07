+++
date = "2017-09-04T23:32:27+08:00"
title = "跨系统超级键鼠"
showonlyimage = false
image = "/img/blog/dual-os-6-soft-kvm/computer_desk.png"
draft = false
weight = 67
+++

使用一套鼠标键盘操作多个系统
<!--more-->

## 魔法键鼠

推荐一个工具 [synergy](https://symless.com/synergy)，它能把一套鼠标和键盘在多个系统间共享，比如你有一台装有 Windows 的笔记本 (W)和二个台式机:一个装有 ArchLinux  (A)，一个装有 MacOS (M)。

你刚坐下就在 W 上收到一封问题上报邮件，里面的报错信息不并常见，需要到 A 上打开 IDE 查找这个特定的报错信息，之后也许会调试代码，那还得查看一下在 M 上 Safari 的网页渲染效果，再回到 W 回复邮件。虽然 A、W、M 三个显示器都整齐的摆放在工作台上，但上述工作还是需要通过它们各自的键盘鼠标完成。

而有了 synergy，你可以手指完全不离开笔记本鼠标键盘，就完成横跨这三个系统的任务。

## 安装配置

Synergy 在 github 上有[源码](https://github.com/symless/synergy)，使用 GPL v2 版权。一方面你可以按照其官方[wiki](https://github.com/symless/synergy/wiki/Compiling)搭建环境自行编译。另一方面，这个 [fork](https://github.com/brahma-dev/synergy-stable-builds) 貌似更靠谱，但我也没有选择自行编译，而是直接使用[编译好](https://www.brahma.world/synergy-stable-builds/)的二进制 Windoes 系统下的 x64 安装包。

无论如何你需要先把 synergy 安装在这三台系统上，并以服务器模式运行在其中一个系统上(比如 W)，而在其他系统上(A 和 M)都以客户端模式运行。记得将它们的主机名全部都写在各自本地解析文件中。

{{< highlight bash >}}
$ sudo pacman -S synergy
$ sudo vi /etc/hosts
{{< /highlight >}}

<br />

### 服务器配置

服务器模式需要运行在那台共享的键盘鼠标的主机上，比如我共享笔记本的键盘鼠标，启动 Synergy 后就勾选服务器模式，首次你需要进入交互设置，配置一下两台主机的相对位置，比如 A 、W、M 分列左中右。这样当你将鼠标移动到 W 屏幕的左边继续移动，鼠标和键盘就开始对 A 系统生效。

选择将屏幕位置和共享偏好等都保存在 synergy.sgc。然后通过定义 Windows Service 或 bat 文件的方式将下面启动指令固化到系统中。

{{< highlight console >}}
C:\Program Files\Synergy+\bin\synergys.exe  -f --debug ERROR \
  --log c:\windows\synergy.log
  -c C:/windows/synergy.sgc
  --address xx.xx.xx.xx:24800
{{< /highlight >}}

<br />

### 客户端配置

其他主机 W、A 都以客户端模式运行，可以打开命令行，直接运行，可以定义为 systemd 的用户级别的 service。

{{< highlight console >}}
$ synergyc -f server-host-name  ## foreground

$ cat ~/.config/systemd/user/synergyc.service
[Unit]
Description=Synergy Client Daemon
After=network.target

[Service]
ExecStart=/usr/bin/synergyc --no-daemon server-name

[Install]
WantedBy=multi-user.target

$ systemctl --user start synergyc

{{< /highlight >}}

<br />

### 更多功能

当服务器程序和运行在各个主机上的客户端连接成功后，你就可使用一套键鼠无缝的在各个屏幕间切换了。不但如此，你还可以启用机器间的共享剪贴板。据说付费版还有像跨机器拖拽文件，还有通信加密等高级功能。19 美元终身使用。值得推荐。

类似软件还有一个 [Input Director](http://www.inputdirector.com/index.html)，免费，但仅能在多个 Windows 系统间共享键鼠。我没有尝试。

### 参考文档

> - ArchLinux wiki [Synergy](https://wiki.archlinux.org/index.php/Synergy)
> - wikipedia [Synergy-(software)](https://en.wikipedia.org/wiki/Synergy_(software))

封面图片来自 [Computer Desk](https://dribbble.com/shots/1613357-Computer-Desk) <a href="https://dribbble.com/slaterdesign"><i class="fa fa-dribbble" aria-hidden="true"></i> Nick Slater</a>  
