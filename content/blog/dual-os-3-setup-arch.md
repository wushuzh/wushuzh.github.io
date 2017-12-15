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

> 如果希望 pacman 安装进度条像吃豆人一样，在 /etc/pacman.conf 的 Options 中增加 ILoveCandy 选项

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
$ sudo systemctl start smbd
$ sudo systemctl enable smbd
Created symlink /etc/systemd/system/multi-user.target.wants/smbd.service → /usr/lib/systemd/system/smbd.service.
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

> 刚安装好 Gnome 后，自带的 gnome-terminal 无法启动。但后来又好了，不清楚是不是和折腾了一下 UTF 还是安装了中文字体相关？反正现在已经正常启动，无法重现，具体原因待有缘后再查了  
>         
我现在用基于 python 的 [Guake](https://wiki.archlinux.org/index.php/Guake) 替代 Gnome-terminal ，也一样是 drop-down-terminal 。叫 Guake 是因为它是受游戏《雷神之锤 Quake 》中显示方式启发开发的: 默认用 F12 唤出。对于选项设定:

1. 通用: 解除 Use VTE titles for tab names，使用快捷键 Ctrl + F2 对当前 tab 命令。 
2. 显示: 将调色盘设定为 Solarized Dark 以便和 vim 插件[vim-colors-solazied](https://vimawesome.com/plugin/vim-colors-solarized-ours)配合

### 安装位于 AUR 的软件

如果你想定制 GNOME 的桌面[主题，图标等](https://wiki.archlinux.org/index.php/GNOME#Appearance)，那其大部分安装包都被标识为 AUR。但是安装 AUR 软件是使用 Archlinux 尝鲜其他各种新软件的一个必备技能。一定要学会。(更为便捷的安装工具，比如```pacaur```或```yaourt```官方都不正式支持，而新手应该先用熟 makepkg，之后也许开始试试 [Cower](https://github.com/falconindy/cower) )

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

确认安装成功——列出所有已安装在系统中的 AUR 软件包
{{< highlight console >}}
$ pacman -Qm
cower 17-2
gnome-shell-extension-arch-update 23-1
gnome-shell-extension-coverflow-alt-tab 1.4-1
gnome-shell-extension-dash-to-dock 61-1
gnome-shell-extension-drop-down-terminal 23-1
gnome-shell-extension-system-monitor-git 801.746f33d-1
google-chrome 61.0.3163.100-1
numix-icon-theme-git 0.r1947.dc833c839-1
oni 0.2.8-1
pyenv 1.1.3-1
pyenv-virtualenv 1:1.1.1-2
pyenv-virtualenvwrapper 20140609-1
vertex-themes 20170128-1
xcursor-human 0.6-4

# remove pkg from AUR is as same as official ones
$ pacman -R pkgname-to-be-removed
{{< /highlight >}}

安装 AUR 上的软件时，也许会遇到了 GPG 验证错误(比如安装 cower 时)，这时需要向系统[导入公钥](https://unix.stackexchange.com/questions/361213/unable-to-add-gpg-key-with-apt-key-behind-a-proxy)。

{{< highlight console >}}
...
==> Verifying source file signatures with gpg...
    cower-17.tar.gz ... FAILED (unknown public key 1EB2638FF56C0C53)
==> ERROR: One or more PGP signatures could not be verified!

$ gpg --recv-keys \
    --keyserver hkp://pgp.mit.edu
    --keyserver-options http-proxy=http://127.0.0.1:xxxx
    1EB2638FF56C0C53
key 1EB2638FF56C0C53:
26 signatures not checked due to missing keys
gpg: key 1EB2638FF56C0C53: public key "Dave Reisner <d@falconindy.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1

# continue install cower

# check aur updates
$ cower -vdu
{{< /highlight >}}

### 关于 keyring

你每次打开 Chrome 等应用时，系统可能都会提示你输入 keyring 密码。按照 [GNOME/Keyring Passwords are not remembered](https://wiki.archlinux.org/index.php/GNOME/Keyring#Passwords_are_not_remembered) 可以了解更多，并设定为无密码提示。

> GPG(GNU Privacy Guard) 使用 PGP 协议 (Pretty Good Privacy) 的开源工具。和《三体》不同，它依靠的不是猜疑链，而是信任链: The Web of Trust。  
> [简单说就是](https://www.reddit.com/r/GnuPG/comments/6tkcnq/eli5_whats_key_signing_and_howwhy_to_sign_a_key/) 当老师检查前一天要家长签字的卷子时，一眼看穿了你的假签名，就是因为他们在家长会的时候交换了彼此的墨宝。而当你把病假条带给校长时，虽然他不认识你爸妈的签名，但他转而通过你的班主任求证，这样只需验证班主任的签名就知道假条是否为仿造。

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
> - reddit [What is the recommended way to update packages installed by the AUR? (And how do you do it?)](https://www.reddit.com/r/archlinux/comments/2kgkfb/what_is_the_recommended_way_to_update_packages/)


封面图片来自 [Two Easy Steps](https://dribbble.com/shots/2266220-Two-Easy-Steps) <a href="https://dribbble.com/iandickens"><i class="fa fa-dribbble" aria-hidden="true"></i> Ian Dickens</a>  
