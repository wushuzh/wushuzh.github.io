+++
date = "2019-10-28T21:31:42+08:00"
title = "ARTS 2019w44"
image = "/img/blog/arts-71-per-week/"
draft = true
weight = 1943
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip



## Share

之前写过 MBR 先装 Windows 再装 archlinux 的双系统安装实践。

而当下绝大多数 PC 机都预装 Windows 10 并使用 GPT 分区表，OEM 厂商为了为方便帮助用户恢复系统，磁盘也被预置为多个分区。这种前置条件下的双系统安装配置实践更加普适性。下面是我的实践记录：

第一阶段：Windows 下的准备工作：

1.0 运行 `msinfo32.exe` [确认 BIOS 模式为 UEFI](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#Windows_UEFI_vs_BIOS_limitations)；

1.1 启动 Windows 的磁盘管理命令行工具 `diskpart`，再次确认 GPT 的使用，减小分区，为 Linux 安装预留空间

1.2 运行 `powercfg.cpl` 关闭[“快速启动”](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#Fast_Start-Up)特性;

1.3 下载最新的 [archlinux 镜像](https://www.archlinux.org/download/)，使用 Rufus 工具制作 U 盘安装介质，这个工具的好处时可以对 U 盘制作多个分区，区分放置引导镜像和其他数据；

1.4 修改注册表，[使 Windows 使用 UTC 时间](https://wiki.archlinux.org/index.php/System_time#UTC_in_Windows)，然后确认 Windows 下的时钟同步激活，使其自动设置时间、时区；

1.5. 重启进入 BIOS ，[关闭 UEFI Secure Boot](https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#UEFI_Secure_Boot) 并将 U 盘启动置于硬盘之上，确认时钟设置；

1.6. 插入第 3 步制作的 U 盘，测试是否能成功引导至命令行

第二阶段：Arch Linux 安装配置

2.1 打通网络，设置网络时钟同步

2.2 为 arch linux 划分磁盘并挂载，注意已有 EFI 分区不需要重建，直接挂载即可：

    举例，为 archlinux 准备了 500 G空余空间，通过 parted 工具做如下设置：
      - `/dev/sda8` 使用 60 G，文件系统 ext4，作为 root 分区 (`/mnt`)
      - `/dev/sda9` 使用 4 G ，作为 swap 分区
      - `/dev/sda10` 使用 400 G，文件系统 ext4，作为 home 分区 (`/mnt/home`)
      - `/dev/sda11` 使用 70 G, 文件系统 fat32，作为和 windows 的共享数据区 (`/mnt/greenzone`)

2.3 在线安装软件

pacstrap /mnt base linux linux-firmware vim

2.4 生成 fstab，chroot 并做主机名、语言，时区设置

2.5 安装、设置 bootloader


封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>