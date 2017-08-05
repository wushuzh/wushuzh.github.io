+++
date = "2017-08-01T19:12:33+08:00"
title = "安装失败，咋整"
showonlyimage = false
image = "/img/blog/deal-with-ks-failure/kickstart-car.png"
topImage = "/img/blog/deal-with-ks-failure/kickstart-car.gif"
draft = false
weight = 45
+++

如何提问以及大公司如何调查问题
<!--more-->

## 问题上报

之前[启动错误，咋整？]要重装系统，结果 Kickstart 安装时遇到了存储问题。所报问题如下：


## 问题分析

Kickstart 安装中利用的应该也是 screen 或 tmux 这样的终端仿真器，同时支持多个终端连接，各自都记录不同类别的日志。所以除了报错时的界面，也可以通过 Ctrl + Alt + F<n> 来查看各自的日志，或通过 shell 进入当前环境中做一些调试和日志收集的工作。

- F1
- F2
- F3
...

报错信息为：

进入 F2 查看系统中磁盘情况： 相关命令

此系统自己有2块 SATA 盘，6块 SSD 固态硬盘，然后通过 4 个 P812 磁盘阵列控制器，外接了 4个 MSA ??? 存储，总计 48 块 SATA 盘。外部连线就挺复杂的，因此磁盘众多，为了能给系统同时提供较高的可靠性和性能，采用了硬件 RAID 和 软件 RAID 结合的方式。

## 网上搜索

没有和我特别一样的错误，但相关的几个如下：

一则有可能是 anaconda 的问题，这不是第一次
二则有的建议是用磁盘 uuid SW RAID 分区配置，我们这边ks文件也是这么做的
...



## 求援外部

给 REDHAT 开 ticket 求助

如何提问

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
LD3: ${DISKS_LIST_EXTERNAL[0]},${DISKS_LIST_EXTERNAL[1]} raid=0
LD4: ${DISKS_LIST_EXTERNAL[12]},${DISKS_LIST_EXTERNAL[13]} raid=0
LD5: ${DISKS_LIST_EXTERNAL[24]},${DISKS_LIST_EXTERNAL[25]} raid=0
LD6: ${DISKS_LIST_EXTERNAL[36]},${DISKS_LIST_EXTERNAL[37]} raid=0
LD7: ${DISKS_LIST_EXTERNAL[9]},${DISKS_LIST_EXTERNAL[10]} raid=1
LD8: ${DISKS_LIST_EXTERNAL[21]},${DISKS_LIST_EXTERNAL[22]} raid=1
LD9: ${DISKS_LIST_EXTERNAL[33]},${DISKS_LIST_EXTERNAL[34]} raid=1
LD10: ${DISKS_LIST_EXTERNAL[45]},${DISKS_LIST_EXTERNAL[46]} raid=1

LD11: ${DISKS_LIST_EXTERNAL[3]},${DISKS_LIST_EXTERNAL[4]},${DISKS_LIST_EXTERNAL[5]},${DISKS_LIST_EXTERNAL[6]},${DISKS_LIST_EXTERNAL[7]},${DISKS_LIST_EXTERNAL[8]} raid=1+0

LD12: ${DISKS_LIST_EXTERNAL[15]},${DISKS_LIST_EXTERNAL[16]},${DISKS_LIST_EXTERNAL[17]},${DISKS_LIST_EXTERNAL[18]},${DISKS_LIST_EXTERNAL[19]},${DISKS_LIST_EXTERNAL[20]} raid=1+0

LD13: ${DISKS_LIST_EXTERNAL[27]},${DISKS_LIST_EXTERNAL[28]},${DISKS_LIST_EXTERNAL[29]},${DISKS_LIST_EXTERNAL[30]},${DISKS_LIST_EXTERNAL[31]},${DISKS_LIST_EXTERNAL[32]} raid=1+0

LD14: ${DISKS_LIST_EXTERNAL[39]},${DISKS_LIST_EXTERNAL[40]},${DISKS_LIST_EXTERNAL[41]},${DISKS_LIST_EXTERNAL[42]},${DISKS_LIST_EXTERNAL[43]},${DISKS_LIST_EXTERNAL[44]} raid=1+0

LD15: ${DISKS_LIST_EXTERNAL[11]},${DISKS_LIST_EXTERNAL[2]} raid=1
LD16: ${DISKS_LIST_EXTERNAL[23]},${DISKS_LIST_EXTERNAL[14]} raid=1
LD17: ${DISKS_LIST_EXTERNAL[35]},${DISKS_LIST_EXTERNAL[26]} raid=1
LD18: ${DISKS_LIST_EXTERNAL[47]},${DISKS_LIST_EXTERNAL[38]} raid=1

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
