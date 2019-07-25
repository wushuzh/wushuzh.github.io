+++
date = "2019-04-08T15:57:50+08:00"
title = "ARTS 2019w15"
showonlyimage = false
image = "/img/blog/arts-42-per-week/server.webp"
draft = false
weight = 1915
tags = ["ARTS"]
+++

Linux 存储相关知识总体概述
<!--more-->

## Algorithm

[leetcode #947 Most Stones Removed with Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/) 给定一些点，如果任意两点的横坐标或纵坐标相同，可删除其中一个点，求最多可以去除多少个给定点。

Union Find 的关键依然是将给定问题转化为 Union find 的数据结构，最直接的方法当然是按题目字面要求，对两个点做连接操作。

{{< highlight python "linenos=inline, hl_lines=6" >}}
def rm_stones_using_union_find(stones: List[List[int]]) -> int:
    uf = UnionFind()
    for x, y in stones:
        for i, j in stones:
            if x == i or y == j:
                uf.union(str(x) + "-" + str(y), str(i) + "-" + str(j))

    return len(uf.p) - len({uf.find_root(str(x) + "-" + str(y)) for x, y in stones})
{{< /highlight >}}

> 提交上述做法时，竟出现了 Time Limit Exceeded （执行超时）的情况。当时使用了字符串格式化语法，不知道是不是类似操作太过耗时耗力？

首先可能的优化：根据题目给定的坐标轴的最大范围，用更简单的算术计算将横纵坐标合并在一起，比如 x * 10000 + y。

但是一个新视角：任何一个给定点(x, y) 是用来连接两个横纵坐标索引的：即坐标索引为最基本单元，轮询每个点时执行 union 操作即可。最终代码一次循环即可：

{{< highlight python "linenos=inline, hl_lines=4 6">}}
def rm_stones_using_union_find(stones: List[List[int]]) -> int:
    uf = UnionFind()
    for x, y in stones:
        uf.union(x, 10000 + y)

    return len(stones) - len({uf.find_root(i) for i,_ in stones})
{{< /highlight >}}

注意比较两种方法最后返回冗余点个数的不同算法。

## Review

本周学习了 [Linux Under the Hood](https://learning.oreilly.com/live-training/courses/linux-under-the-hood/0636920257462) 存储相关的章节——存储作为一个重要外设，掌握基本知识十分必要，先上一张来自 [Thomas-Krenn Wiki](https://www.thomas-krenn.com/en/wiki/Linux_Storage_Stack_Diagram) 的存储全栈图。

{{< figure src="https://upload.wikimedia.org/wikipedia/commons/3/30/IO_stack_of_the_Linux_kernel.svg" title="The Linux Storage Stack Diagram" >}}

虽未能在上图出现，但在计算机启动中起重要作用的部分是 [Boot sector(引导扇区)](https://en.wikipedia.org/wiki/Boot_sector) 分为:

- [MBR(主引导分区)](https://en.wikipedia.org/wiki/Master_boot_record) 中可用于记录分区的空间有限（4 个）：一般是 3 个主分区，1 个扩展分区（内含多个逻辑分区），管理磁盘的最大空间为 2 TB, 一般用 fdisk 管理分区；
- [GPT(全局唯一标识分区表)](https://en.wikipedia.org/wiki/GUID_Partition_Table) 中预留了更多可用于记录分区的空间 (最高 128 个)，但OS 或 device driver 层面会对最终支持数量做进一步限制，比如 Linux 的 Major（内核分给每类磁盘驱动程序）/Minor Number（驱动程序分给每个设备），一般用 gdisk 管理磁盘分区；
- 当 lsblk 不可用时，可以通过 `cat /proc/partitions` 查看块设备和分区信息，更极客的做法是通过以下命令查看磁盘引导区 `xxd -l 512 /dev/sda`

---

*图顶端* ：和应用进程直接相关的是 *VFS* 和 *FS* 。据维基百科的不完全统计，100余种文件系统被发明和使用在各类操作系统中。而 [VFS(虚拟文件系统)](https://en.wikipedia.org/wiki/Virtual_file_system) 则是 Linux 内核为上层应用提供了一致的系统调用接口来访问多种文件系统。

这些不同文件系统在不同的技术条件和时代背景下产生和不断演化，各具特点，比如下表中的几个常见的文件系统被 Linux 内核直接支持，ext系列、xfs 和 btrfs；而通过 VFS 中的 [FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) 接口可使用那些仅能存在于用户空间的文件系统（或因许可协议和内核不符，或因功能太过超前），比如 [ZFS](https://en.wikipedia.org/wiki/ZFS)、[SSHFS](https://en.wikipedia.org/wiki/SSHFS)、NTFS-3G、GlusterFS

|       | indexing | journal | extents | cow |
|-------|----------|---------|---------|-----|
| ext2  | h-tree   | -       | -       | -   |
| ext3  | h-tree   | +       | -       | -   |
| ext4  | h-tree   | +       | +       | -   |
| xfs   | b-tree   | +       | -       | -   |
| btrfs | b-tree   | +       | +       | +   |

---

*图中部* ：为了更为统一的管理组织越来越多的磁盘设备，操作系统提供了 [Device mapper 框架](https://en.wikipedia.org/wiki/Device_mapper) 。在这个基础上，又出现了 [LVM(逻辑卷管理)](https://en.wikipedia.org/wiki/Logical_volume_management)、[SW RAID](https://en.wikipedia.org/wiki/RAID#Software-based)、[LUKS(Linux Unified Key Setup)](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup) 等各具特色的存储方案。

---

[损坏文件的修复](https://wiki.archlinux.org/index.php/Identify_damaged_files)是一个略微“复杂”和“危险”的操作。不同文件系统提供了各自的工具，对存储底层技术有更深的理解能保证我们不盲目行动，一些重要的概念包括:

- [磁盘扇区](https://en.wikipedia.org/wiki/Disk_sector) 和 [逻辑区块地址](https://en.wikipedia.org/wiki/Logical_block_addressing) 分配
- 磁盘管理框架 Device mapper 和 LVM 配置管理
- 文件的数据结构 [inode](https://en.wikipedia.org/wiki/Inode) 

比如 StackExchange 上的这个问题 
[What is the relationship of inodes, LBA, Logical Volumes, blocks, and sectors?](https://unix.stackexchange.com/questions/106861/what-is-the-relationship-of-inodes-lba-logical-volumes-blocks-and-sectors)
```
               .-----------------.
               |      inode      |
               '-----------------'
                        |
               .-----------------.
               |      EXT4       |
               '-----------------'
                        |
             .---------------------.
             | logical volume (LV) | --- part of LVM
             '---------------------'
                        |
              .-------------------.
              | volume group (VG) |  --- part of LVM
              '-------------------'
                        |
                .---------------.
                | /dev/<device> |
                '---------------'
                        |
       .--------------------------------.
       | Logical Block Addressing (LBA) |
       '--------------------------------'
                        |
               .-----------------.
               | blocks/sectors  |
               '-----------------'
                        |
                       HDD     
                    _.-----._  
                  .-         -.
                  |-_       _-|
                  |  ~-----~  |
                  |           |
                  `._       _.'
                     "-----"   
```

最后对于[闪存](https://en.wikipedia.org/wiki/Flash_memory)，因为其记忆损耗的特性，需要用专门的[文件系统](https://en.wikipedia.org/wiki/Flash_file_system)做[损耗平衡](https://en.wikipedia.org/wiki/Wear_leveling)以延长其使用寿命，比如 Ubifs, JFFS, YAFFS 。

## Tip

借助 [asciiflow](http://asciiflow.com/) 可以用ASCII字符画简易图示：

- POSIX 系统中的 inode 
- iSCSI 的设置图示
  1. 在 Target 设置 devname baz `/backstore/block/`
  2. 在 Target 设置 wwn = iqn.yyyy-mm.domain:tfoo `/iscsi`
  3. 在 Initiator 找出 iqn （ibar) `/etc/iscsi/initiatorname.iscsi`
  4. 在 Target 配置 ACL 允许 Initiator wwn (ibar） `/iscsi/iqnfoo/tpg1/acl`
  5. 在 Target 配置 LUNS 配置共享设备 baz
  6. 在 Target 上启动 target 服务
  7. 在 Initiator 发现 Target LUN
  8. 在 Initiator 登陆 Target
  9. 在 Initiator 可以看到共享设备

```
           symlink
              +
              |
              v
2nd name   name
hardlink   hardlink
    +         +
    |         |
    |         v
    |      +--+--+
    +----->+inode| Everything but name
           +--+--+
              |
              |
              v
          +-------+
          |1|2|3|4|
          +-------+

```
```
service:   +----------+      +----------+
target     |----------|      |----+     |
           ||sda||sdb||      ||sda|     | tool:
tool:      +-------+--+      +----+     | iscsiadm
targetcli  |       v  |      |          | lsscsi
           |      LUN |      |          |
           |          |      |          |
           |    SAN   |      |          |
           |          |      |          +----+
           |Target    |      |Initiator ||sdb|
           +----------+      +-----+---------+
             TCP|3260              |
                <------------------+
                      discovery    |
                <------------------+
                      login

```

## Share

{{< figure src="/img/blog/arts-42-per-week/spark.webp" title="Spark MLib now and future" >}}


拓展阅读：

> - [不完全文件系统对比表 (Comparison of file systems)](https://en.wikipedia.org/wiki/Comparison_of_file_systems)
> - btrfs 被 SUSE 使用了很长时间，为其用户提供了一些很特别的功能，比如 [subvolume 和 snapshot](https://en.wikipedia.org/wiki/Btrfs#Subvolumes_and_snapshots) 。然而 Redhat 表示 RHEL 8 中会放弃 btrfs [why redhat abandon btrfs where SUSE makes it default.?](https://access.redhat.com/discussions/3138231)
> - [ISCSI Basics](https://www.thomas-krenn.com/en/wiki/ISCSI_Basics)


封面图片来自 [Server](https://dribbble.com/shots/976148-Server) <a href="https://dribbble.com/zht"><i class="fa fa-dribbble" aria-hidden="true"></i> Zhivko Terziivanov</a>