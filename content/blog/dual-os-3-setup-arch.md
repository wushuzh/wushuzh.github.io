+++
date = "2017-08-15T17:37:07+08:00"
title = "Archlinux 配置"
showonlyimage = false
image = "/img/blog/dual-os-3-setup-arch/setup-Arch.png"
draft = false
weight = 64
+++

从 0 到 1 设定 Archlinux 桌面
<!--more-->

## 配置网络、用户

配置完下面这些，就可以暂时告别 root ，转而通过普通用户(wushuzh)执行各种操作了
{{< highlight console >}}
# ### setup wifi as needed
# systemctl enable dhcpcd@enp0s25.service
# systemctl start dhcpcd@enp0s25.service
# ip a

# pacman -S openssh
# ### edit /etc/ssh/sshd_config as needed
# systemctl enable sshd.socket
# systemctl start sshd.socket

# useradd -m -g users -G audio,kvm,storage,wheel -s /bin/bash wushuzh
# passwd wushuzh

# pacman -S sudo
# visudo to uncomment the line for "%wheel ALL ..."
{{< /highlight >}}

<br />

### 参考文档

> - Tofeeq (2017-01-19) [Easy installing arch linux UEFI dual boot with windows](https://lampjs.wordpress.com/2017/01/19/easy-installing-arch-linux-dual-boot-with-windows-uefi-or-mbr-for-beginners/)


封面图片来自 [Two Easy Steps](https://dribbble.com/shots/2266220-Two-Easy-Steps) <a href="https://dribbble.com/iandickens"><i class="fa fa-dribbble" aria-hidden="true"></i> Ian Dickens</a>  
