+++
date = "2017-08-15T17:37:07+08:00"
title = "Archlinux 配置"
showonlyimage = false
image = "/img/blog/dual-os-3-setup-arch/setup-Arch.png"
draft = false
weight = 64
+++

从 0 到 1 设定 Archlinux 桌面环境
<!--more-->

后面还要安装不少软件，所以现在是优化软件镜像源的好时机。

首先，访问[镜像列表生成器](https://www.archlinux.org/mirrorlist/)，选择你所在地区，生成一个可用镜像的列表。然后执行下载速度的[排序](https://wiki.archlinux.org/index.php/Mirrors#Sorting_mirrors)。

{{< highlight console >}}
# cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
# sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup
# rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup \
        > /etc/pacman.d/mirrorlist
{{< /highlight >}}

<br />

## 添加用户

即便你是这台机器的唯一用户，带着 root 权限做所有的日常操作总不合适。

正确姿势是创建一个你的专属用户：

- 加入 wheel 组，只在必要时通过 sudo 提升权限
- 通过 visudo 配置 sudo ，比如提升权限的同时，使用当前环境变量 https_proxy
- 加入 audio 组，管理声音
- 加入 kvm 组，管理虚拟机
- 加入 storage 组，管理 U 盘

配置完下面这些，就可以暂时告别 root ，转而通过普通用户(wushuzh)执行各种操作了

{{< highlight console >}}
# useradd -m -g users -G audio,kvm,storage,wheel -s /bin/bash wushuzh
# passwd wushuzh

# pacman -S sudo
# visudo to uncomment the line for "%wheel ALL ..."
#    /etc/sudoers
#    Defaults env_keep += "ftp_proxy http_proxy https_proxy no_proxy"
{{< /highlight >}}

<br />

## 远程连接

我需要能从笔记本连到这个台式机上，所以 ssh 的配置当然不能缺。

{{< highlight console >}}
# ### setup wifi as needed
$ sudo systemctl enable dhcpcd@enp0s25.service
$ systemctl start dhcpcd@enp0s25.service
$ ip a

$ sudo pacman -S openssh
# ### edit /etc/ssh/sshd_config as needed
# ### enable/start sshd.service if lots of connections
$ sudo systemctl enable sshd.socket  
$ sudo systemctl start sshd.socket

# ### conf Encrypted SOCKS tunnel
# ### conf user level unit service
{{< /highlight >}}

除了 sftp 传输文件，samba 应该是更为方便的传输模式。

{{< highlight console >}}
$ sudo pacman -S samba
$ sudo cp /etc/samba/smb.conf.default /etc/samba/smb.conf
...
$ sudo smbpasswd -a wushuzh
{{< /highlight >}}

<br />

## 配置声卡显卡

安装```alsa-utils```并测试声音

之前用 gpu-z 查看过我的显卡。为了能用上英伟达自己的专有显卡驱动，需要做如下这些工作。

{{< highlight console >}}
$ lspci -k | grep -A 2 -E "(VGA|3D)"
18:00.0 VGA compatible controller: NVIDIA Co. G86 [Quadro NVS 290]...
        Subsystem: NVIDIA Corporation Device 0492
        Kernel driver in use: nvidia
{{< /highlight >}}

到英伟达官网上查看这块显卡的 code name 或是其所属家族，然后选择安装对应驱动安装包。

{{< highlight console >}}
$ sudo pacman -S nvidia-340xx
$ sudo reboot
{{< /highlight >}}

> 看好 Nvidia 的股票，人工智能时代，这种卖硬件的厂商应该能赚大钱。截止到发稿前 2017-08-16 股价为 166.98 刀。从今年一月到现在涨了 50~60%  
> 而 AMD 的前景应该也不错，单股价格也便宜，13.02 刀。留此存照，看一年后能涨多少。

<br />

## 图形化桌面

安装配置 GNOME 下载的时间比较久，但安装配置并不算太困难。

{{< highlight console >}}
$ sudo pacman -S gnome gnome-extra
$ sudo pacman -S xorg-xinit
$ sudo pacman -S base-devel
{{< /highlight >}}

### 安装位于 AUR 的软件

如果你想定制 GNOME 的桌面[主题，图标等](https://wiki.archlinux.org/index.php/GNOME#Appearance)，那其大部分安装包都被标识为 AUR。但是安装 AUR 软件是使用 Archlinux 尝鲜其他各种新软件的一个必备技能。一定要学会。(更为便捷的安装工具，比如```yaourt```官方都不正式支持)

{{< highlight console >}}
$ cd /tmp
$ git clone https://aur.archlinux.org/vertex-themes.git
$ cd vertex-themes/
$ makepkg -si

$ cd /tmp
$ git clone https://aur.archlinux.org/google-chrome.git
$ cd google-chrome/
$ makepkg -si
{{< /highlight >}}

### 参考文档

> - Tofeeq (2017-01-19) [Easy installing arch linux UEFI dual boot with windows](https://lampjs.wordpress.com/2017/01/19/easy-installing-arch-linux-dual-boot-with-windows-uefi-or-mbr-for-beginners/)
> - Arch wiki [Secure Shell # SOCKS tunnel](https://wiki.archlinux.org/index.php/Secure_Shell#Encrypted_SOCKS_tunnel)
> - Arch wiki [Sudo conf and tips](https://wiki.archlinux.org/index.php/Sudo#Environment_variables)
> - Arch wiki [Samba Server conf](https://wiki.archlinux.org/index.php/Samba#Creating_usershare_path)
> - Arch wiki [Advanced Linux Sound Architecture](https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture#Unmuting_the_channels)
> - Arch wiki [Nvidia Install](https://wiki.archlinux.org/index.php/NVIDIA#Installation)
> - Arch wiki [GNOME # Xorg sessions](https://wiki.archlinux.org/index.php/GNOME#Xorg_sessions)
> - Arch wiki [xinit](https://wiki.archlinux.org/index.php/Xinit)
> - Arch wiki [Arch User Repo](https://wiki.archlinux.org/index.php/Arch_User_Repository#Installing_packages)


封面图片来自 [Two Easy Steps](https://dribbble.com/shots/2266220-Two-Easy-Steps) <a href="https://dribbble.com/iandickens"><i class="fa fa-dribbble" aria-hidden="true"></i> Ian Dickens</a>  
