+++
date = "2017-07-19T18:45:13+08:00"
title = "这咋整——存储失效"
showonlyimage = false
image = "/img/blog/blog-using-hugo-1/desktop.png"
draft = true
weight = 30
tags = [ "Howto", "Storage", "Linux" ]
+++

如何从存储类错误中恢复。
<!--more-->

为了说明提供长期可靠存储的困难，先和大家一起回忆一下 《三体 Ⅲ 死神永生》尾声的一个片段

主干背景：程圣母带领人类痛失威慑纪元……但勤劳勇敢的地球人不屈不挠，抱着死也要拉个垫背的坚强信念，利用最后的引力波向全宇宙大喊出了三体星系的祖坟位置……
掩体纪元：三体星系的祖坟如约被刨，但对太阳系的黑暗森林打击也紧随而至……
谈话地点：冥王星地球文明博物馆
谈话人物：罗辑、程心等
谈话重点：罗辑介绍了在这里这么苦逼的为人类文明刻碑，就是因为信息对不同的介质，存储的效率和时限不能兼得：

- 普通量子存储器 500 年
- U盘和硬盘 5000 年
- 特殊金属光盘 10 万年
- 特种合成纸张和油墨 20 万年

科学家团队最终确认了将信息保存一亿年的唯一可行方法。大概是因为得到这个结论还花了不少经费，所以宣布的时候多少还些点扭捏：“ 把字都刻在石头上 ”

回到现实，大概最日常硬件故障就是磁盘损坏了吧？

当下很火热的大数据应用，普遍都是将数据做分布式的冗余存储 ( 如 HDFS )。而就算那些非大数据应用，无论是系统的生产环境部署，或对重要数据的存储也都会做硬/软件的 RAID ，以便提供更好的数据可靠性和冗余度。比如在主板上集成或外加一块扩展卡（比如 HP 410i），或使用外置存储(如 HP P2000 和 MSA ) 上的存储控制器，将多块物理磁盘按 RAID1，RAID5 等方式组合为一个逻辑磁盘后，再对外服务。此种模式下更新一块坏盘就很容易。不开箱直接插拔问题磁盘即可，更新后，硬件存储控制器会自动将原有数据按冗余设置在新盘上同步好，无需人工干预。

上面这都是日常实验室维护，但最近我遇到问题都是来自服务器迁移——换机房后发生的。

标准创建 RAID10 的方法 https://www.tecmint.com/create-raid-10-in-linux/


[root@nio120 dev]# cat /proc/mdstat
Personalities : [raid0] [raid1]
md6 : active raid0 sdp1[0] sdh1[2] sdl1[1] sdd1[3]
      4394500096 blocks super 1.1 512k chunks

md5 : inactive sds1[0](S) sdo1[1](S) sdk1[2](S) sdg1[3](S)
      1464306688 blocks super 1.1

md24 : active raid1 sda6[2] sdb6[0]
      206381056 blocks super 1.2 [2/2] [UU]
      bitmap: 1/2 pages [4KB], 65536KB chunk

md22 : active raid1 sda5[2] sdb5[0]
      10728448 blocks super 1.2 [2/2] [UU]

md3 : active raid1 sda2[0] sdb2[1]
      21456768 blocks super 1.1 [2/2] [UU]

md20 : active raid1 sda1[2] sdb1[0]
      53653504 blocks super 1.2 [2/2] [UU]

md1 : active raid1 sda3[0] sdb3[1]
      524224 blocks super 1.0 [2/2] [UU]

unused devices: <none>

[root@nio120 dev]# mdadm --detail /dev/md5
/dev/md5:
        Version : 1.1
     Raid Level : raid0
  Total Devices : 4
    Persistence : Superblock is persistent

          State : inactive

           Name : nio120:5  (local to host nio120)
           UUID : cb622579:188d8cef:0c91003b:04e85a9e
         Events : 24562603

    Number   Major   Minor   RaidDevice

       -       8       97        -        /dev/sdg1
       -       8      161        -        /dev/sdk1
       -       8      225        -        /dev/sdo1
       -      65       33        -        /dev/sds1


---

---

[root@nio120 ~]# mdadm -E /dev/sd[gkos]1
/dev/sdg1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
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
/dev/sdk1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
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
    Device UUID : a55812c0:c4beff84:d6236c75:8a1b6d64

Internal Bitmap : 8 sectors from superblock
    Update Time : Wed Jul 27 18:07:15 2016
       Checksum : 64aca313 - correct
         Events : 3408

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 2
   Array State : A.AA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sdo1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
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
    Device UUID : 1c0e9a86:eddcc56f:b243204d:82a91aea

Internal Bitmap : 8 sectors from superblock
    Update Time : Wed Jul 27 18:07:10 2016
       Checksum : 735f2476 - correct
         Events : 3398

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 1
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sds1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
  Creation Time : Tue Feb  2 10:28:01 2016
     Raid Level : raid10
   Raid Devices : 4

 Avail Dev Size : 1171791872 (558.75 GiB 599.96 GB)
     Array Size : 585606144 (558.48 GiB 599.66 GB)
  Used Dev Size : 585606144 (279.24 GiB 299.83 GB)
    Data Offset : 262144 sectors
   Super Offset : 0 sectors
   Unused Space : before=262072 sectors, after=586185728 sectors
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

---
[root@nio120 dev]# cat /etc/mdadm.conf
ARRAY /dev/md/20  metadata=1.2 UUID=5f81b47d:d454ccc3:f7fdb8d9:dbd962e6 name=nio120:20
ARRAY /dev/md/3  metadata=1.1 UUID=4c4a66ec:858bfe9a:56bbc4ec:cb9c5594 name=nio120:3
ARRAY /dev/md/1  metadata=1.0 UUID=754daa5e:2ae8ad99:3c01ebe6:37db6b65 name=nio120:1
ARRAY /dev/md/22  metadata=1.2 UUID=e9fd3c20:1400219c:1d4b5b73:5866e841 name=nio120:22
ARRAY /dev/md/24  metadata=1.2 UUID=00db712e:faa2c0aa:6209c4f9:bee9050e name=nio120:24
ARRAY /dev/md/6  metadata=1.1 UUID=37c8bec8:d720bec2:ba752cc6:063aea59 name=nio120:6
ARRAY /dev/md/5  metadata=1.1 UUID=cb622579:188d8cef:0c91003b:04e85a9e name=nio120:5


---
[root@nio120 dev]# mdadm --stop /dev/md5
mdadm: stopped /dev/md5

[root@nio120 dev]# mdadm --assemble --scan
mdadm: /dev/md/5 assembled from 1 drive - not enough to start the array.

[root@nio120 dev]# mdadm --assemble /dev/md5 /dev/sd[gkos]1
mdadm: /dev/sdg1 is busy - skipping
mdadm: /dev/sdk1 is busy - skipping
mdadm: /dev/sdo1 is busy - skipping
mdadm: /dev/sds1 is busy - skipping

[root@nio120 ~]# mdadm --manage /dev/md5 --stop
mdadm: stopped /dev/md5

[root@nio120 ~]# mdadm --assemble --force /dev/md5 /dev/sd[gkos]1
mdadm: forcing event count in /dev/sds1(0) from 24562577 upto 24562603
mdadm: forcing event count in /dev/sdk1(2) from 3408 upto 24562603
mdadm: clearing FAULTY flag for device 1 in /dev/md5 for /dev/sdk1
mdadm: Marking array /dev/md5 as 'clean'
mdadm: /dev/md5 has been started with 3 drives (out of 4).

---


[root@nio120 temp]#  mdadm --detail /dev/md5
/dev/md5:
        Version : 1.1
  Creation Time : Tue Feb  2 10:28:01 2016
     Raid Level : raid10
     Array Size : 585606144 (558.48 GiB 599.66 GB)
  Used Dev Size : 292803072 (279.24 GiB 299.83 GB)
   Raid Devices : 4
  Total Devices : 3
    Persistence : Superblock is persistent

  Intent Bitmap : Internal

    Update Time : Wed Jul 19 16:45:09 2017
          State : clean, degraded
 Active Devices : 3
Working Devices : 3
 Failed Devices : 0
  Spare Devices : 0

         Layout : near=2
     Chunk Size : 512K

           Name : nio120:5  (local to host nio120)
           UUID : cb622579:188d8cef:0c91003b:04e85a9e
         Events : 24562649

    Number   Major   Minor   RaidDevice State
       0      65       33        0      active sync set-A   /dev/sds1
       -       0        0        1      removed
       2       8      161        2      active sync set-A   /dev/sdk1
       3       8       97        3      active sync set-B   /dev/sdg1


---

[root@nio120 temp]# mdadm -E /dev/sd[gkos]1
/dev/sdg1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
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
    Update Time : Wed Jul 19 16:49:45 2017
       Checksum : adbd6d3a - correct
         Events : 24562841

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 3
   Array State : A.AA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sdk1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
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
    Device UUID : a55812c0:c4beff84:d6236c75:8a1b6d64

Internal Bitmap : 8 sectors from superblock
    Update Time : Wed Jul 19 16:49:45 2017
       Checksum : 67f9f7b2 - correct
         Events : 24562841

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 2
   Array State : A.AA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sdo1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
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
    Device UUID : 1c0e9a86:eddcc56f:b243204d:82a91aea

Internal Bitmap : 8 sectors from superblock
    Update Time : Wed Jul 27 18:07:10 2016
       Checksum : 735f2476 - correct
         Events : 3398

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 1
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sds1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
  Creation Time : Tue Feb  2 10:28:01 2016
     Raid Level : raid10
   Raid Devices : 4

 Avail Dev Size : 1171791872 (558.75 GiB 599.96 GB)
     Array Size : 585606144 (558.48 GiB 599.66 GB)
  Used Dev Size : 585606144 (279.24 GiB 299.83 GB)
    Data Offset : 262144 sectors
   Super Offset : 0 sectors
   Unused Space : before=262072 sectors, after=586185728 sectors
          State : clean
    Device UUID : cc52449a:dbbc9518:ae736e28:c5424d37

Internal Bitmap : 8 sectors from superblock
    Update Time : Wed Jul 19 16:49:45 2017
       Checksum : 7e94df00 - correct
         Events : 24562841

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 0
   Array State : A.AA ('A' == active, '.' == missing, 'R' == replacing)

---

mdadm --manage /dev/md5 --stop
mdadm --assemble --scan

http://bitsofmymind.com/2014/03/14/how-to-fix-fsck-your-root-file-system-that-you-have-to-boot-into-on-linux/

http://dev.bizo.com/2012/07/mdadm-device-or-resource-busy.html

https://www.thomas-krenn.com/en/wiki/Mdadm_recover_degraded_Array

https://anders.com/cms/411/Linux/Software.RAID/inactive/mdadm
watch -n .5 cat /proc/mdstat

[root@nio120 temp]# mdadm --detail /dev/md5
/dev/md5:
        Version : 1.1
  Creation Time : Tue Feb  2 10:28:01 2016
     Raid Level : raid10
     Array Size : 585606144 (558.48 GiB 599.66 GB)
  Used Dev Size : 292803072 (279.24 GiB 299.83 GB)
   Raid Devices : 4
  Total Devices : 4
    Persistence : Superblock is persistent

  Intent Bitmap : Internal

    Update Time : Fri Jul 21 13:11:15 2017
          State : active, degraded, recovering
 Active Devices : 3
Working Devices : 4
 Failed Devices : 0
  Spare Devices : 1

         Layout : near=2
     Chunk Size : 512K

 Rebuild Status : 59% complete

           Name : nio120:5  (local to host nio120)
           UUID : cb622579:188d8cef:0c91003b:04e85a9e
         Events : 24570204

    Number   Major   Minor   RaidDevice State
       0      65       33        0      active sync set-A   /dev/sds1
       4       8      225        1      spare rebuilding   /dev/sdo1
       2       8      161        2      active sync set-A   /dev/sdk1
       3       8       97        3      active sync set-B   /dev/sdg1
[root@nio120 temp]# mdadm -E /dev/sd[sokg]1
/dev/sdg1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
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
    Update Time : Fri Jul 21 13:11:46 2017
       Checksum : adbef9f6 - correct
         Events : 24570217

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 3
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sdk1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
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
    Device UUID : a55812c0:c4beff84:d6236c75:8a1b6d64

Internal Bitmap : 8 sectors from superblock
    Update Time : Fri Jul 21 13:11:46 2017
       Checksum : 67fb846e - correct
         Events : 24570217

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 2
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sdo1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x3
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
  Creation Time : Tue Feb  2 10:28:01 2016
     Raid Level : raid10
   Raid Devices : 4

 Avail Dev Size : 585607168 (279.24 GiB 299.83 GB)
     Array Size : 585606144 (558.48 GiB 599.66 GB)
  Used Dev Size : 585606144 (279.24 GiB 299.83 GB)
    Data Offset : 262144 sectors
   Super Offset : 0 sectors
Recovery Offset : 348517120 sectors
   Unused Space : before=262064 sectors, after=1024 sectors
          State : clean
    Device UUID : 6bd85aca:b066f56f:6d9cfa0b:d470ff42

Internal Bitmap : 8 sectors from superblock
    Update Time : Fri Jul 21 13:11:46 2017
  Bad Block Log : 512 entries available at offset 72 sectors
       Checksum : e7286d4c - correct
         Events : 24570217

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 1
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
/dev/sds1:
          Magic : a92b4efc
        Version : 1.1
    Feature Map : 0x1
     Array UUID : cb622579:188d8cef:0c91003b:04e85a9e
           Name : nio120:5  (local to host nio120)
  Creation Time : Tue Feb  2 10:28:01 2016
     Raid Level : raid10
   Raid Devices : 4

 Avail Dev Size : 1171791872 (558.75 GiB 599.96 GB)
     Array Size : 585606144 (558.48 GiB 599.66 GB)
  Used Dev Size : 585606144 (279.24 GiB 299.83 GB)
    Data Offset : 262144 sectors
   Super Offset : 0 sectors
   Unused Space : before=262072 sectors, after=586185728 sectors
          State : clean
    Device UUID : cc52449a:dbbc9518:ae736e28:c5424d37

Internal Bitmap : 8 sectors from superblock
    Update Time : Fri Jul 21 13:11:46 2017
       Checksum : 7e966bbc - correct
         Events : 24570217

         Layout : near=2
     Chunk Size : 512K

   Device Role : Active device 0
   Array State : AAAA ('A' == active, '.' == missing, 'R' == replacing)
