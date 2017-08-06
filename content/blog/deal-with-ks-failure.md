+++
date = "2017-08-01T19:12:33+08:00"
title = "安装失败，咋整"
showonlyimage = false
image = "/img/blog/deal-with-ks-failure/kickstart-car.png"
topImage = "/img/blog/deal-with-ks-failure/kickstart-car.gif"
draft = false
weight = 45
+++

如何提问以及大公司如何查问题
<!--more-->

## 问题上报

之前 “[启动错误，咋整？]({{< relref "blog/deal-with-boot-failure.md" >}})”里提到的服务器终于要重装操作系统，之前听说大型服务器有重大变动，比如重启，重装，升级，搬移都最好要先看黄历、拜菩萨、挑日子，因为容易出事。而这台很长时间不曾重装过的机器在安装时果然遇到了问题。所报问题如下：

{{< highlight console >}}
Starting installer, one monent...
anaconda 21.40.22.93-1 for Red Hat Enterprise Linux 7.3 started.
* installation iog files are stored in /tmp during the installation
* shell is available on TTY2
* when reporting a bug add logs from /tmp as separate text/plain attachncnts
91:51:09 Running pre-installation scripts
99:56:44 Not asking for VNC because of an automated install
99:56:44 Not asking for VNC because text mode was explicitly asked for in kickstart
99:56:44 Not asking for VNC because we don't have a network
Starting automated install............
Checking software selection
Generating updated storage configuration
storage configuration failed: Unable to allocate requested partition scheme.
===========================================================================
===========================================================================
Installation

1) [x] Language settings         2) [x] Time settings
       (English (United States))        (Asia/Shanghai timezone)
3) Lx] Installation source       4) [x] Software selection
       (Local media)                    (Custom software selected)
5) [!] Installation Destination  6) [x] Xdump
       (No disks selected)              (Kdump is enabled)
7) [ ] Network configuration     8) [ ] User creation
       (Not connected)                  (No user will be created)
Not enough space in file systems for the current software selection. An additional 1793.04 NiB is needed.
Enter 'b' to ignore the warning and attempt to install anyway.
Please make your choice from above ['q' to quit | 'b' to begin installation | 'r' to refresh]:


[anacoud] 1:main* 2:shell 3:log 4:storage-log 5:program-log L. Switch tab: Alt+Tab | Help: F1

{{< /highlight >}}


## 问题分析

因为操作系统需要为自家产品做各种定制，为了达到部署的高效，安装使用了 Kickstart 。anancoda 作为安装程序，尤其是其 TUI 的交互界面，是使用了同时支持多个终端连接的仿真器 tmux ，将各个终端分别记录不同类别的日志。

因此当出现报错时，用户可以通过 Ctrl + Alt + F<n> 来跳转到不同的终端，查看各方面的日志，也可进入 shell 环境中做一些基本的调试和日志打包收集工作。

！！！
- F1
- F2
- F3
...

报错信息为：
进入 F！！！ 查看存储方面的最新日志：

进入 F2 查看系统中磁盘情况： 相关命令

此系统自己有2块 SATA 盘，6块 SSD 固态硬盘，然后通过 4 个 P812 磁盘阵列控制器，外接了 4个 MSA ??? 存储，总计 48 块 SATA 盘。外部连线就挺复杂的，因此磁盘众多，为了能给系统同时提供较高的可靠性和性能，采用了硬件 RAID 和 软件 RAID 结合的方式。

<img alt="ext lds" src="/img/blog/deal-with-ks-failure/ext-lds.png" class="img-responsive">

## 网上搜索

没有和本文完全一致的错误，有几个相关的如下：

有的处理建议比较简单……不相关

有的建议是用磁盘 uuid SW RAID 分区配置，我们这边ks文件也是这么做的

！！！有可能是 anaconda 的问题，虽然是大厂出品，但仍不排除这个问题来自 anaconda 或 pyblivet

## 求援外部

我决定让 RedHat 来帮助查看一下这个问题。大的方面来说，我们公司作为其付费订阅会员，有困难找他们咨询合情合理，RedHat 一般或是 anaconda 源码拥有者，或是其方案整合方，他们的视野和对技术理解的深度将能极大加速问题处理的进程。而对个人来说，这是一个非常重要的学习机会。

首先，通读相关用户手册后，应该在 Google 或 Redhat 知识库中查找看这是不是已经被提出过，一个成熟软件的很多问题都源于用户对工具错误理解、不当配置或使用造成的。只有基本理解，并确认过没有相同或类似问题，或已经尝试了自己能力所及的各种调试后，才好咨询外部专家。

然后，就是通过其问题上报系统，撰写问题描述，交互讨论问题。这部分做的好坏对问题处理的进度影响很大。比如，

- 能否将自己的业务逻辑很好剥离，找出问题的核心？
- 对与自身业务逻辑结合很密切的问题，如何才能最高效的帮外部专家理解当下的使用场景？
- 能否正确执行外部专家给你的调查步骤，并将结果做高效反馈？

这还是在《聪明地提问》


之所以这么大张旗鼓的说了很多如何提问的建议，是觉得这是一个工程师非常重要的素质。从结果来看，凡是提问前做很多思考的同学，不但自身技术提高的快，而且也应该能得到其所在组织和同事的认可、肯定。

## BIOS 中的错误

HP

2 internal disk 300GB
6 internal SSD 120GB
48 external disk
12 * 600GB
36 * 300GB

https://www.tecmint.com/ip-command-examples/

https://www.howtogeek.com/118337/stupid-geek-tricks-change-your-ip-address-from-the-command-line-in-linux/

http://www.thegeekstuff.com/2009/11/how-to-execute-ping-command-only-for-n-number-of-packets/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+TheGeekStuff+(The+Geek+Stuff)

LD0: DISKS_LIST_INTERNAL[0] raid 0
LD1: DISKS_LIST_INTERNAL[1] raid=0
LD2:

 DISKS_LIST_SSD_INTERNAL[0]},${DISKS_LIST_SSD_INTERNAL[1]},${DISKS_LIST_SSD_INTERNAL[2]},${DISKS_LIST_SSD_INTERNAL[3]},${DISKS_LIST_SSD_INTERNAL[4]},${DISKS_LIST_SSD_INTERNAL[5] raid=1+0

[root@nio120 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0         51G  2.5G   46G   6% /
tmpfs           127G     0  127G   0% /dev/shm
/dev/md4        194G   60M  184G   1% /alcatel
/dev/md6        2.7T   73M  2.6T   1% /alcatel/backup
/dev/md5        550G   70M  522G   1% /alcatel/temp
/dev/md1        488M   43M  421M  10% /boot
/dev/md2         10G  114M  9.4G   2% /var


[root@nio120 ~]# lsblk
NAME    MAJ:MIN RM   SIZE RO TYPE   MOUNTPOINT
sr0      11:0    1  1024M  0 rom
sda       8:0    0 279.4G  0 disk
├─sda1    8:1    0  51.2G  0 part
│ └─md0   9:0    0  51.2G  0 raid1  /
├─sda2    8:2    0  20.5G  0 part
│ └─md3   9:3    0  20.5G  0 raid1  [SWAP]
├─sda3    8:3    0   512M  0 part
│ └─md1   9:1    0   512M  0 raid1  /boot
├─sda4    8:4    0     1K  0 part
├─sda5    8:5    0  10.2G  0 part
│ └─md2   9:2    0  10.2G  0 raid1  /var
└─sda6    8:6    0   197G  0 part
  └─md4   9:4    0 196.8G  0 raid1  /alcatel
sdb       8:16   0 279.4G  0 disk
├─sdb1    8:17   0  51.2G  0 part
│ └─md0   9:0    0  51.2G  0 raid1  /
├─sdb2    8:18   0  20.5G  0 part
│ └─md3   9:3    0  20.5G  0 raid1  [SWAP]
├─sdb3    8:19   0   512M  0 part
│ └─md1   9:1    0   512M  0 raid1  /boot
├─sdb4    8:20   0     1K  0 part
├─sdb5    8:21   0  10.2G  0 part
│ └─md2   9:2    0  10.2G  0 raid1  /var
└─sdb6    8:22   0   197G  0 part
  └─md4   9:4    0 196.8G  0 raid1  /alcatel
sdd       8:48   0 558.7G  0 disk
└─sdd1    8:49   0 558.7G  0 part
  └─md6   9:6    0   2.7T  0 raid0  /alcatel/backup
sdc       8:32   0 335.3G  0 disk
└─sdc1    8:33   0 335.3G  0 part
sde       8:64   0 279.4G  0 disk
└─sde1    8:65   0 279.4G  0 part
sdf       8:80   0 838.1G  0 disk
└─sdf1    8:81   0 838.1G  0 part
sdg       8:96   0 279.4G  0 disk
└─sdg1    8:97   0 279.4G  0 part
  └─md5   9:5    0 558.5G  0 raid10 /alcatel/temp
sdh       8:112  0 558.7G  0 disk
└─sdh1    8:113  0 558.7G  0 part
  └─md6   9:6    0   2.7T  0 raid0  /alcatel/backup
sdj       8:144  0 838.1G  0 disk
└─sdj1    8:145  0 838.1G  0 part
sdi       8:128  0 279.4G  0 disk
└─sdi1    8:129  0 279.4G  0 part
sdl       8:176  0 558.7G  0 disk
└─sdl1    8:177  0 558.7G  0 part
  └─md6   9:6    0   2.7T  0 raid0  /alcatel/backup
sdm       8:192  0 279.4G  0 disk
└─sdm1    8:193  0 279.4G  0 part
sdk       8:160  0 279.4G  0 disk
└─sdk1    8:161  0 279.4G  0 part
  └─md5   9:5    0 558.5G  0 raid10 /alcatel/temp
sdp       8:240  0   1.1T  0 disk
└─sdp1    8:241  0   1.1T  0 part
  └─md6   9:6    0   2.7T  0 raid0  /alcatel/backup
sdn       8:208  0 838.1G  0 disk
└─sdn1    8:209  0 838.1G  0 part
sdo       8:224  0 279.4G  0 disk
└─sdo1    8:225  0 279.4G  0 part
  └─md5   9:5    0 558.5G  0 raid10 /alcatel/temp
sds      65:32   0 558.9G  0 disk
└─sds1   65:33   0 558.9G  0 part
  └─md5   9:5    0 558.5G  0 raid10 /alcatel/temp
sdr      65:16   0   1.7T  0 disk
└─sdr1   65:17   0   1.7T  0 part
sdq      65:0    0 558.9G  0 disk
└─sdq1   65:1    0 558.9G  0 part



## anaconda 操作

## 回传日志

## 磁盘分析

## 如何提问

## 社区示例 ks 文件

封面图片来自 [Kickstart Car](https://dribbble.com/shots/2342802-Kickstart-Car) <a href="https://dribbble.com/KristianDuffy"><i class="fa fa-dribbble" aria-hidden="true"></i> Kristian Duffy</a>  


http://sg.danny.cz/scsi/lsscsi.html

collect log
https://access.redhat.com/solutions/20358


https://serverfault.com/questions/391439/how-can-i-troubleshoot-a-failing-linux-kickstart-installation

https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-simple-install-kickstart.html

https://access.redhat.com/solutions/1233263
https://access.redhat.com/solutions/1258933


https://github.com/CentOS/Community-Kickstarts
