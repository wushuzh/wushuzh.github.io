+++
date = "2017-08-10T18:48:26+08:00"
title = "双系统 2: Linux"
showonlyimage = false
image = "/img/blog/dual-os-2-install-linux/linux-m.png"
draft = true
weight = 63
+++

选择哪个 Linux 版本？是个问题
<!--more-->

## Linux 版本

{{< figure src="/img/blog/dual-os-2-install-linux/linux-flavour.jpg" title="Decision Tree" >}}

上面的决策树送给犹豫不决的你。我为了促使自己不断地折腾，同时显得比较硬核，依然选择 Arch Linux。

## 制作 U 盘

[官网](https://www.archlinux.org/download/)下载安装介质 ：我下载的是 2017.08.01 的版本，内核 4.12.3，ISO 大小 516 MB。
等待下载的同时，开个浏览器阅读 [USB flash installation media](https://wiki.archlinux.org/index.php/USB_flash_installation_media#In_Windows), 然后从中挑一种顺眼的方法——我选 dd for Windows——将安装介质写入 U 盘。

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

<br />

## 引导模式

先上个双保险——就算 Linux 安装失败——仍能回到你的 Windows 避风港，这里再次确认当前的引导模式。

- 使用```diskpart```，详见["双系统 1: 微软视窗"]({{< relref "blog/dual-os-1-install-win.md" >}}) ；
- 使用```msinfo32```，选择根节点“系统摘要”，查看右侧面板中的 BIOS 模式的值，如果为"传统"或"Legacy"，就是 BIOS-MBR 模式，若为 UEFI，就是 UEFI-GPT 模式。

然后根据这个信息和当前 Windows 版本反复阅读 [ArchLinux 双启动 wiki](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows)的相关章节——里面全是知识点: chainload、bootloader、GRUB、GRUB Legacy、Syslinux、UEFI、BIOS、GPT、MBR——务必做到谋定而后动。

## 关于时钟

之前已经为 Windows 设置了 ntp 同步，为了之后双系统的时间互不干扰，应该统一将硬件系统时间(BIOS 中)和 Windows 系统时间(通过注册表修改)都配为与 UTC 配合。详见 [Time std wiki](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#Time_standard)

## 动手安装

BIOS 设好 U 盘启动，进入系统(命令行终端)开始做安装前配置。值得一提的是系统提供多个虚拟终端间供你自由切换，按下 ``` Alt + 方向键 ``` ，设好代理，启动一个文本型的 Web 浏览器查看安装在线文档。最好只看英文，图片当然也不行。

{{< highlight bash >}}
export http_proxy=YOUR_PROXY
elinks wiki.archlinux.org/index.php/Installation_guide
{{< /highlight >}}

### 磁盘分区

之前已经预留了 100+ GB 的未分区空间用于 Linux 安装。```fdisk -l```查看确认。已安装的 Windows 8 占用了

- /dev/sda1 350M (Type: HPFS/NTFS/exFAT id: 7)
- /dev/sda2 150G 对应 C 盘 (同上)
- /dev/sda3 38G 对应 D 盘 (同上)

MBR 上的 3 个主分区都已被占用，Linux 必须通过扩展分区实现，通过```fdisk /dev/sda``` 尝试如下分区:

#### 经典分区

1. 新建扩展分区 /dev/sda4 含所有未分配空间 (Type: Extended id: 5)
2. 新建逻辑分区 /dev/sda5 80 GB (Type: Linux id: 83)
3. 新建交换分区 /dev/sda6 4 GB (Type: Linux swap / Solaris id: 82)
4. 新建双系统共享区 /dev/sda7 20 GB (Type: W95 FAT32 (LBA) id: c)

#### LVM 分区



### 格式化分区

#### 经典分区

{{< highlight bash >}}
mkfs.ext4 /dev/sda5
mkswap /dev/sda6
mkfs.vfat /dev/sda7
{{< /highlight >}}

#### LVM 分区



### 挂载分区

#### 经典分区

{{< highlight bash >}}
mount /dev/sda5 /mnt
swapon /dev/sda6
mkdir /mnt/greenzone
mount -t vfat /dev/sda7 /mnt/greenzone
{{< /highlight >}}

## 安装

所有软件必须通过网络下载，修改其镜像文件```/etc/pacman.d/mirrorlist```，将自己所在国家的地址放在最上边就可以了。

{{< highlight bash >}}
# 安装核心软件
pacstrap /mnt base

# 生成文件系统挂载表
genfstab -U /mnt >> /mnt/etc/fstab

# 切换到新安装到硬盘的文件系统上
arch-chroot /mnt

# 配置时间
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc

# 配置本地化字符
vi /etc/locale.gen
locale-gen

# 配置主机名
/etc/hostname
/etc/hosts  

# 配置 root 密码
passwd
{{< /highlight >}}

### 经典分区

{{< highlight bash >}}
pacman -S grub
......
grub-install --target=i386-pc /dev/sda6 --force

{{< /highlight >}}

### LVM 分区

### 参考文档

> - Ashwin Kailas (2016-11-16) [Arch Linux (LVM) dual-boot with Windows tutorial](https://medium.com/@ashwinkailas/arch-linux-lvm-dual-boot-with-windows-tutorial-e327ea8f3140) 文中涉及更先进的 EFI 引导模式，但我参考了除此之外的其他部分

封面图片来自 [Introduction to Linux](https://dribbble.com/shots/1862256-Introduction-to-Linux) <a href="https://dribbble.com/kabojanowska"><i class="fa fa-dribbble" aria-hidden="true"></i> Kasia</a>  
