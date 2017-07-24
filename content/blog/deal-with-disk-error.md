+++
date = "2017-07-22T20:14:43+08:00"
title = "遇到磁盘错误，咋整？"
showonlyimage = false
image = "/img/blog/deal-with-disk-error/hw-repair.png"
draft = false
weight = 41
tags = [ "Howto", "Storage", "Linux" ]
+++

磁盘错误导致进入紧急模式，该如何恢复？
<!--more-->

当前大数据应用的存储架构很少再采用 [RAID](https://en.wikipedia.org/wiki/RAID) ——此领域最常听到的是 JBOD ，隶属于[Non-RAID drive architectures](https://en.wikipedia.org/wiki/Non-RAID_drive_architectures)

但本文涉及的老系统不但使用了 [磁盘冗余阵列](https://zh.wikipedia.org/wiki/RAID)，而且不是在硬件层面(智能存储阵列控制器)上直接做 RAID，而使用的是 SW-RAID，并且不是普通 RAID，而是[嵌套 RAID](https://en.wikipedia.org/wiki/Nested_RAID_levels) …… 怎么看都挺复杂的。

### 案例 C 磁盘受损

问题描述：这是一台用于模拟客户生产环境的服务器，一年多没重装，仅通过常规产品升级维护着。上电后，据说遇到了磁盘错误，被强制进入维护模式，之后管理员将问题分区从 fstab 中注释掉，系统可以正常启动，但在修复磁盘和恢复软 RAID 遇到了麻烦。

---  

- 首先 /proc/mdstat 查看出问题的冗余阵列分区，处于 inactive 状态，由四个分区组成。  

> 另有配置文件 /etc/mdadm.conf

{{< highlight console >}}
[root@hostC ~]# cat /proc/mdstat |grep -A 3 md5
md5 : inactive sds1[0](S) sdo1[1](S) sdk1[2](S) sdg1[3](S)
      1464306688 blocks super 1.1

[root@hostC ~]#
{{< /highlight >}}

---

- mdadm 命令查看此冗余阵列的磁盘分区详情，RAID Level 为 raid0，查看产品设计文档，此处应该为 raid10 ，即需要重新将下属 4 个分区做 RAID 组装。

{{< highlight console >}}
[root@hostC ~]# mdadm --detail /dev/md5
/dev/md5:
        Version : 1.1
     Raid Level : raid0
  Total Devices : 4
    Persistence : Superblock is persistent

          State : inactive

           Name : hostC:5  (local to host hostC)
           UUID : cb622579:188d8cef:0c91003b:04e85a9e
         Events : 24562603

    Number   Major   Minor   RaidDevice

       -       8       97        -        /dev/sdg1
       -       8      161        -        /dev/sdk1
       -       8      225        -        /dev/sdo1
       -      65       33        -        /dev/sds1
{{< /highlight >}}

---

- 检查( -E --examine )下属各个磁盘的状态，几个下属块设备也都印证了他们应为 raid10 ，但各个盘的 event 标号都不一致，更新时间有的追溯到很早，另外各个磁盘的状态也都不是 AAAA，猜测这块磁盘已经带病工作很久了，这次搬家，运输过程中一颠一震，再也坚持不下去，撂挑子休息了……

{{< highlight console >}}
[root@hostC ~]# mdadm -E /dev/sd[gkos]1
/dev/sdg1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : hostC:5  (local to host hostC)
  Creation Time : Tue Feb  2 10:28:01 2016
     Raid Level : raid10
   Raid Devices : 4

 Avail Dev Size : 585607168 (279.24 GiB 299.83 GB)
     Array Size : 585606144 (558.48 GiB 599.66 GB)
  Used Dev Size : 585606144 (279.24 GiB 299.83 GB)
    Data Offset : 262144 sectors
   Super Offset : 0 sectors
   Unused Space : before=262072 sectors, after=1024 sectors
          State : clean
    Device UUID : 9941486b:d67ae4a0:f36d875e:eea1faf9

Internal Bitmap : 8 sectors from superblock
    Update Time : Sat Jul 15 15:10:19 2017
       Checksum : adb90efa - correct
         Events : 24562603

         Layout : near=2
     Chunk Size : 512K

 Device Role : Active device 3
 Array State : A..A ('A' == active, '.' == missing, 'R' == replacing)

{{< /highlight >}}
{{< highlight console >}}
 /dev/sdk1:

             ... : ...
           State : clean
     Device UUID : a55812c0:c4beff84:d6236c75:8a1b6d64

 Internal Bitmap : 8 sectors from superblock
     Update Time : Wed Jul 27 18:07:15 2016
        Checksum : 64aca313 - correct
          Events : 3408

          Layout : near=2
      Chunk Size : 512K

  Device Role : Active device 2
  Array State : A.AA ('A' == active, '.' == missing, 'R' == replacing)

{{< /highlight >}}

{{< highlight console >}}
 /dev/sdo1:

             ... : ...
           State : clean
     Device UUID : 1c0e9a86:eddcc56f:b243204d:82a91aea

 Internal Bitmap : 8 sectors from superblock
     Update Time : Wed Jul 27 18:07:10 2016
        Checksum : 735f2476 - correct
          Events : 3398

          Layout : near=2
      Chunk Size : 512K

  Device Role : Active device 1
  Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)

{{< /highlight >}}

{{< highlight console >}}
 /dev/sds1:

             ... : ...
           State : clean
     Device UUID : cc52449a:dbbc9518:ae736e28:c5424d37

 Internal Bitmap : 8 sectors from superblock
     Update Time : Sat Jul 15 15:10:05 2017
        Checksum : 7e908098 - correct
          Events : 24562577

          Layout : near=2
      Chunk Size : 512K

  Device Role : Active device 0
  Array State : A..A ('A' == active, '.' == missing, 'R' == replacing)
{{< /highlight >}}

---

- 先将相应冗余阵列停掉，再重新组装。

{{< highlight console >}}

[root@hostC ~]# mdadm --stop /dev/md5
mdadm: stopped /dev/md5

[root@hostC ~]# mdadm --assemble --scan
mdadm: /dev/md/5 assembled from 1 drive - not enough to start the array.

{{< /highlight >}}

这里的报错应该是因为 mdadm 还不够智能(或者不想担责任)，需要我们将其下属磁盘都显示指定，才能继续组装。

{{< highlight console >}}

[root@hostC ~]# mdadm --assemble /dev/md5 /dev/sd[gkos]1
mdadm: /dev/sdg1 is busy - skipping
mdadm: /dev/sdk1 is busy - skipping
mdadm: /dev/sdo1 is busy - skipping
mdadm: /dev/sds1 is busy - skipping

{{< /highlight >}}

手动显式指定了，但又遇到设备正忙的错误。之后我又执行了一次 mdadm 停止命令，之后立即执行组装命令，成功了。但仍然显示仅仅组装上了总共 4 块磁盘中的 3 块。

{{< highlight console >}}

[root@hostC ~]# mdadm --manage /dev/md5 --stop
mdadm: stopped /dev/md5

[root@hostC ~]# mdadm --assemble --force /dev/md5 /dev/sd[gkos]1
mdadm: forcing event count in /dev/sds1(0) from 24562577 upto 24562603
mdadm: forcing event count in /dev/sdk1(2) from 3408 upto 24562603
mdadm: clearing FAULTY flag for device 1 in /dev/md5 for /dev/sdk1
mdadm: Marking array /dev/md5 as 'clean'
mdadm: /dev/md5 has been started with 3 drives (out of 4).
{{< /highlight >}}

---

此时再查看冗余磁盘阵列的状态，已变为 clean degraded 的状态。

{{< highlight console >}}
[root@hostC ~]#  mdadm --detail /dev/md5
/dev/md5:
            ... : ...
     Raid Level : raid10
     Array Size : 585606144 (558.48 GiB 599.66 GB)
  Used Dev Size : 292803072 (279.24 GiB 299.83 GB)
   Raid Devices : 4
  Total Devices : 3
    Persistence : Superblock is persistent
            ... : ...
    Update Time : Wed Jul 19 16:45:09 2017
          State : clean, degraded
 Active Devices : 3
Working Devices : 3
 Failed Devices : 0
  Spare Devices : 0
            ... : ...
           UUID : cb622579:188d8cef:0c91003b:04e85a9e
         Events : 24562649

    Number   Major   Minor   RaidDevice State
       0      65       33        0      active sync set-A   /dev/sds1
       -       0        0        1      removed
       2       8      161        2      active sync set-A   /dev/sdk1
       3       8       97        3      active sync set-B   /dev/sdg1

{{< /highlight >}}

---

而下属的各个磁盘，除 /dev/sdo1 以外，Event # 都已同步。

{{< highlight console >}}
[root@hostC temp]# mdadm -E /dev/sd[gkos]1 \
    | grep [Events|Update|Array State|]

/dev/sdg1:
    Update Time : Wed Jul 19 16:49:45 2017
         Events : 24562841
   Array State : A.AA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sdk1:
    Update Time : Wed Jul 19 16:49:45 2017
         Events : 24562841
   Array State : A.AA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sdo1:
    Update Time : Wed Jul 27 18:07:10 2016
         Events : 3398
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sds1:
    Update Time : Wed Jul 19 16:49:45 2017
         Events : 24562841
   Array State : A.AA ('A' == active, '.' == missing, 'R' == replacing)
{{< /highlight >}}
---

之后将 fstab 中拿掉的行重新拿回，重启服务器，但又一次被强制进入维护模式

{{< highlight console >}}
Welcome to emergency mode! After logging in, type "journalctl -xb" to view
system logs, "systemctl reboot" to reboot, "systemctl default" or ^D to
try again to boot into default mode.
Give root password for maintenance
(or type Crontrol-D to continue):
{{< /highlight >}}

---

按提示进入执行 journalctl 查看日志，发现磁盘需要修复，大概和之前没有装载上的磁盘有点关系？

{{< highlight console >}}
some-timestamp hostC systemd-fsck[16431]:
  /dev/md5: Inode xxx has a bad extended attribute block yyy.
some-timestamp hostC systemd-fsck[16431]:
  /dev/md5: UNEXPECTED INCONSISTENCY: RUN fsck MANUALLY.
some-timestamp hostC systemd-fsck[16431]:
  (i.e., without -a or -p options)
some-timestamp hostC systemd-fsck[16431]:
  Running request emergency.target/start/replace
some-timestamp hostC systemd[1]:
  Started File System Check on /dev/disk/by-uuid/aaa-bbb-ccc-ddd-......
{{< /highlight >}}

因为系统已经处于维护模式，之后的修复相对简单。可以参考 Antoine (2014-03-14) [how-to-fix-fsck-your-root-file-system-that-you-have-to-boot-into-on-linux]( http://bitsofmymind.com/2014/03/14/how-to-fix-fsck-your-root-file-system-that-you-have-to-boot-into-on-linux/)

其中一个可能遇到的问题是 fsck 命令相应设备已经挂载，无法继续执行的错误，可以将相应报错的设备或挂机点重挂载为只读，然后再执行磁盘检测。

{{< highlight console >}}
[root@hostC ~]# df -h |grep md5
/dev/md5    550G    56G    467G    11%    /somefolder
[root@hostC ~]# fsck /dev/md5
e2fsck 1.42.9 (28-Dec-2013)
/dev/md5 is mounted.
e2fsck: Cannot continue, aborting.

[root@hostC ~]# cat /proc/mounts |grep md5
/dev/md5 /somefolder ext4 rw,nodev,relatime,stripe=256,data=ordered 0 0
[root@hostC ~]# mount -o remount,ro /somefolder
[root@hostC ~]# cat /proc/mounts |grep md5
/dev/md5 /somefolder ext4 ro  ,nodev,relatime,stripe=256,data=ordered 0 0
[root@hostC ~]# fsck -y /dev/md5
...
{{< /highlight >}}

修复完毕后，将冗余阵列组装为 4 块设备，重启服务器，顺利进入系统。
可以执行命令 ```watch -n .5 cat /proc/mdstat``` 每隔 0.5 秒监控一下冗余阵列的同步状态。

{{< highlight console >}}
[root@hostC ~]# mdadm --detail /dev/md5
/dev/md5:
            ... : ...
          State : active, degraded, recovering
 Active Devices : 3
Working Devices : 4
 Failed Devices : 0
  Spare Devices : 1

Rebuild Status : 59% complete

    Number   Major   Minor   RaidDevice State
       0      65       33        0      active sync set-A   /dev/sds1
       4       8      225        1      spare rebuilding   /dev/sdo1
       2       8      161        2      active sync set-A   /dev/sdk1
       3       8       97        3      active sync set-B   /dev/sdg1

[root@hostC ~]# mdadm -E /dev/sd[sokg]1 \
    | grep [dev|Array State]
/dev/sdg1:
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sdk1:
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sdo1:
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sds1:
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
{{< /highlight >}}

缩略语解释


另外，TECMINT 上有一篇[创建 RAID10 的方法](https://www.tecmint.com/create-raid-10-in-linux/)，有助于理解。

封面图片来自 [Computer repair](https://dribbble.com/shots/2287977-Computer-repair) <a href="https://dribbble.com/Frizler"><i class="fa fa-dribbble" aria-hidden="true"></i> Anton Fritsler</a>  
