+++
date = "2017-08-10T18:48:26+08:00"
title = "双系统 2: Linux"
showonlyimage = false
image = "/img/blog/dual-os-2-install-arch/linux-m.png"
draft = false
weight = 63
+++

选择哪个 Linux 版本？是个问题
<!--more-->

## Linux 版本

{{< figure src="/img/blog/dual-os-2-install-arch/linux-flavour.jpg" title="Decision Tree" >}}

上面的决策树送给犹豫不决的你。为了促使自己不断地折腾，同时显得比较硬核，我依然选择 Arch Linux。

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

先上个双保险——就算 Linux 安装失败——仍能回到 Windows ，再次确认计算机的引导模式。

- 使用```diskpart```，详见["双系统 1: 微软视窗"]({{< relref "blog/dual-os-1-install-win.md" >}}) ；
- 使用```msinfo32```，选择根节点“系统摘要”，查看右侧面板中的 BIOS 模式的值，如果为"传统"或"Legacy"，就是 BIOS-MBR 模式，若为 UEFI，就是 UEFI-GPT 模式。

然后根据这个信息和当前 Windows 版本反复阅读 [ArchLinux 双启动 wiki](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows) 相关章节——同学们，这里面全是知识点啊: chainload、bootloader、GRUB、GRUB Legacy、Syslinux、UEFI、BIOS、GPT、MBR——反正务必做到谋定而后动。

我的机器非常老了，只能采用 BIOS-MBR 模式。关于分区，[Talk:Dual_boot_with_Windows#MBR-BIOS](https://wiki.archlinux.org/index.php/Talk:Dual_boot_with_Windows#MBR-BIOS) 还提到了另一种实现双启动的办法：即先分区，然后安装 Windows ，之后装 Linux ，并在最后用 grub-install 等工具覆盖 Windows 引导区。这应该也可行——甚至更好——但这需要重新安装 Windows 我这回并没有采用。

## 关于时钟

之前已经为 Windows 设置了 ntp 同步，为了之后双系统的时间互不干扰，应该统一将硬件系统时间(BIOS 中)和 Windows 系统时间(要通过修改注册表实现)都设定为 UTC 配合。详见 [Time std wiki](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#Time_standard)

## 动手安装

BIOS 设好 U 盘启动，进入系统(命令行终端)开始做安装前配置。值得一提的是系统提供多个虚拟终端间供你自由切换，按下 ``` Alt + 方向键 ``` ，设好代理，启动一个文本型的 Web 浏览器查看安装在线文档。最好只看英文，图片当然也看不了。

{{< highlight bash >}}
export http_proxy=YOUR_PROXY
elinks wiki.archlinux.org/index.php/Installation_guide
{{< /highlight >}}

<br />

### 磁盘分区、格式化和挂载

#### 磁盘分区

之前已经预留了 100+ GB 的未分区空间用于 Linux 安装。```fdisk -l```查看确认。已安装的 Windows 8 占用了

- /dev/sda1 350M (Type: HPFS/NTFS/exFAT id: 7)
- /dev/sda2 150G 对应 C 盘 (同上)
- /dev/sda3 38G 对应 D 盘 (同上)

MBR 上的 3 个主分区都已被占用，Linux 必须通过扩展分区实现，通过```fdisk /dev/sda``` 或 parted 工具尝试完成如下分区:

1. 新建扩展分区 /dev/sda4 含所有未分配空间 (Type: Extended id: 5)
2. 新建逻辑分区 /dev/sda5 200 MB (Type: Linux 0x83)
2. 新建逻辑分区 /dev/sda6 80 GB (Type: Linux  0x83)
3. 新建交换分区 /dev/sda7 4 GB (Type: Linux swap / Solaris 0x82)
4. 新建双系统共享区 /dev/sda8 20 GB (Type: W95 FAT32 (LBA) 0x0c)

<br />

#### 文件系统格式化和挂载

{{< highlight bash >}}
# 文件系统格式化
mkfs.ext3 /dev/sda5
mkfs.ext4 /dev/sda6
mkswap /dev/sda7
mkfs.vfat /dev/sda8

# 挂载
mount /dev/sda6 /mnt
mkdir /mnt/boot
mount /dev/sda5 /mnt/boot
swapon /dev/sda7
mkdir /mnt/greenzone
mount -t vfat /dev/sda8 /mnt/greenzone
{{< /highlight >}}

> 后面专门再写一篇如何通过 LVM 进行分区。

### 系统安装和配置

Archlinux 追新：安装的软件都从网上下载最新版：这里修改其镜像文件```/etc/pacman.d/mirrorlist```，将自己所在国家的地址移到最上边就可以了。

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
hostnamectl set-hostname wushuzh
cat /etc/hostname
# exit current session and relogin to chk
/etc/hosts

# 配置键盘布局
localectl set-keymap us
cat /etc/vconsole.conf
localectl set-locale LANG=en_US.UTF-8
cat /etc/locale.conf

# 配置 root 密码
passwd
{{< /highlight >}}

### 引导区设定

这部分是双系统最核心的部分——Linux 和 Windows 的双启动能否成功在此一举。首先，继续上面的 root shell 会话，安装 grub 程序，将启动程序和配置都安装在 /boot (sda5) 上。然后将其关键文件(core.img)设定为不可更改，重新生成 grub.cfg 中相关磁盘的 uuid。将用 /boot 分区的前 512 个字节，输出到共享分区备用。

> 尚不知道是否每次更新 Linux 内核时都需要做这一系列的操作

{{< highlight bash >}}
pacman -S grub

grub-install --target=i386-pc \
    --grub-setup=/bin/true --recheck --debug /dev/sda5
chattr +i /boot/grub/i386-pc/core.img
pacman -S linux
grub-mkconfig -o /boot/grub/grub.cfg

dd if=/dev/sda5 of=/greenzone/linux.bin bs=512 count=1

{{< /highlight >}}

重启回到 Windows 8，打开一个带有管理员权限的命令行，拷贝刚才准备的引导区文件 linux.bin，在使用 Windows 的 bootLoader 更新配置，为其追加一个 Linux 启动项。将其和 GRUB 的 bootloader 级联，达到启动 Linux 的目的。重启，测试 Windows 和 Linux 的引导。

{{< highlight bat >}}
REM copy the file linux.bin from F: to C:
bcdedit /create /d "Linux" /application BOOTSECTOR

bcdedit /set {ID} device partition=c:
bcdedit /set {ID}  path \linux.bin
bcdedit /displayorder {ID} /addlast
bcdedit /timeout 30
{{< /highlight >}}

<br />

### 参考文档

> -  Archlinux wiki [Installation guide](https://wiki.archlinux.org/index.php/Installation_guide)
> - Archlinux wiki [Dual boot with Windows # Using Windows boot loader](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#Using_Windows_boot_loader)
> - Archlinux wiki [GRUB/Tips and tricks # Install to partition disk](https://wiki.archlinux.org/index.php/GRUB/Tips_and_tricks#Install_to_partition_or_partitionless_disk)
> - iceflatline 2009-09-16 [How to Dual Boot Windows 7 and Linux using BCDEdit](https://www.iceflatline.com/2009/09/how-to-dual-boot-windows-7-and-linux-using-bcdedit/)
> - Ashwin Kailas (2016-11-16) [Arch Linux (LVM) dual-boot with Windows tutorial](https://medium.com/@ashwinkailas/arch-linux-lvm-dual-boot-with-windows-tutorial-e327ea8f3140) 文中涉及更先进的 EFI 引导模式，但可参考 LVM 的划分方式

封面图片来自 [Introduction to Linux](https://dribbble.com/shots/1862256-Introduction-to-Linux) <a href="https://dribbble.com/kabojanowska"><i class="fa fa-dribbble" aria-hidden="true"></i> Kasia</a>
