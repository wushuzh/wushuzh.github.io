+++
date = "2019-04-15T15:31:37+08:00"
title = "ARTS 2019w16"
showonlyimage = false
image = "/img/blog/arts-43-per-week/cpu.webp"
draft = true
weight = 1916
tags = ["ARTS"]
+++

待定
<!--more-->

## Algorithm

[leetcode #924 Minimize Malware Spread](https://leetcode.com/problems/minimize-malware-spread/) N 个节点组成的网络，若两个节点有连接，则 graph[i][j] == 1 。给定一些节点会带有病毒，可以通过上述连接关系传播，如果只能从初始的病毒节点去除一个节点，请返回能使得最终网络被感染程度最小的节点索引。

本题标准的 union find 处理，需要特别注意题目最后的提醒，去除某一节点后，要认识到：网络仍可能通过其他的病毒初始节点的冗余链接关系继续感染任何潜在节点。

没有太多技巧，理清思路，逐步处理：

1. 统计网络最终由多少个 group 组成，每个 group 大小；
2. 对每个 group 中病毒源节点的数量进行统计；
3. 只有当某 group 中有且只有一个病毒源时，去除的效果才是有效的；
4. 去除效果应该表述为：此单个病毒源所在的 group 其他节点都不受影响；

逐个分析病毒源节点，找出能独立影响网络的最大源即可。

{{< highlight python >}}
def min_malware_spread_using_union_find(graph: List[List[int]],
                                        initial: List[int]) -> int:
    N = len(graph)
    uf = UnionFind(N)
    for i in range(N):
        for j in range(i):
            if graph[i][j]:
                uf.union(i, j)

    a_roots_count = collections.Counter([uf.find_root(i) for i in uf.p])
    v_roots_count = collections.Counter([uf.find_root(v) for v in initial])

    cnt, idx = 0, min(initial)
    for vid in initial:
        v_root = uf.find_root(vid)
        if v_roots_count[v_root] == 1:
            if a_roots_count[v_root] > cnt:
                cnt, idx = a_roots_count[v_root], vid
            elif a_roots_count[v_root] == cnt:
                idx = min(idx, vid)
    return idx
{{< /highlight >}}

## Review

本周偶然查看服务器的时候，看到处理器信息，主板上二个 Socket ，安装 CPU 的状态为 **Populated, Enabled**，未安装 CPU 的 Socket 状态为 **Unpopulated** 。既然有空余插槽，就想是否能再找一个闲置 CPU 插上，但遍寻后未果。假设真能找到了额外的 CPU 和一些内存条为服务器做一次硬件升级(Scale Up)，要注意的是，如果内存条数量不足无法满配，需要按特定的安插顺序做内存安装。一般打开机器后的盖板背面都有会有内存条安插顺序的说明。类似 HP 的这篇文档 [HP ProLiant Gen8 Servers - Memory Architecture for Intel Xeon E5-2600 Series Processors](https://support.hpe.com/hpsc/doc/public/display?docId=mmr_kc-0109346)

{{< highlight txt "linenos=inline, hl_lines=48 56-58 76">}}
$ sudo dmidecode -t processor
# dmidecode 3.1
Getting SMBIOS data from sysfs.
SMBIOS 2.8 present.

Handle 0x0400, DMI type 4, 42 bytes
Processor Information
        Socket Designation: Proc 1
        Type: Central Processor
        Family: Xeon
        Manufacturer: Intel
        ID: D7 06 02 00 FF FB EB BF
        Signature: Type 0, Family 6, Model 45, Stepping 7
        Flags:
                FPU (Floating-point unit on-chip)
                VME (Virtual mode extension)
                DE (Debugging extension)
                PSE (Page size extension)
                TSC (Time stamp counter)
                MSR (Model specific registers)
                PAE (Physical address extension)
                MCE (Machine check exception)
                CX8 (CMPXCHG8 instruction supported)
                APIC (On-chip APIC hardware supported)
                SEP (Fast system call)
                MTRR (Memory type range registers)
                PGE (Page global enable)
                MCA (Machine check architecture)
                CMOV (Conditional move instruction supported)
                PAT (Page attribute table)
                PSE-36 (36-bit page size extension)
                CLFSH (CLFLUSH instruction supported)
                DS (Debug store)
                ACPI (ACPI supported)
                MMX (MMX technology supported)
                FXSR (FXSAVE and FXSTOR instructions supported)
                SSE (Streaming SIMD extensions)
                SSE2 (Streaming SIMD extensions 2)
                SS (Self-snoop)
                HTT (Multi-threading)
                TM (Thermal monitor supported)
                PBE (Pending break enabled)
        Version:  Intel(R) Xeon(R) CPU E5-2670 0 @ 2.60GHz
        Voltage: 1.4 V
        External Clock: 100 MHz
        Max Speed: 4800 MHz
        Current Speed: 2600 MHz
        Status: Populated, Enabled
        Upgrade: Socket LGA2011
        L1 Cache Handle: 0x0710
        L2 Cache Handle: 0x0720
        L3 Cache Handle: 0x0730
        Serial Number: Not Specified
        Asset Tag: Not Specified
        Part Number: Not Specified
        Core Count: 8
        Core Enabled: 8
        Thread Count: 16
        Characteristics:
                64-bit capable

Handle 0x0401, DMI type 4, 42 bytes
Processor Information
        Socket Designation: Proc 2
        Type: Central Processor
        Family: Xeon
        Manufacturer: Intel
        ID: 00 00 00 00 00 00 00 00
        Signature: Type 0, Family 0, Model 0, Stepping 0
        Flags: None
        Version:
        Voltage: 1.4 V
        External Clock: 200 MHz
        Max Speed: 4800 MHz
        Current Speed: Unknown
        Status: Unpopulated
        Upgrade: Socket LGA2011
        L1 Cache Handle: 0x0716
        L2 Cache Handle: 0x0726
        L3 Cache Handle: 0x0736
        Serial Number: Not Specified
        Asset Tag: Not Specified
        Part Number: Not Specified
        Characteristics:
                64-bit capable

{{< /highlight >}}

可以上面高亮的行看到现有的 CPU 8 核 16 线程，这可能是就是 Intel 的超线程技术？

Intel 的 x86 CPU 为了提升并行计算，曾使用过一项现在看来颇具疑问的技术：[Hyper-threading](https://en.wikipedia.org/wiki/Hyper-threading)，支持此特性的操作系统，可以将单个物理 CPU 视为两个逻辑/虚拟内核：他们有各自独立的[Architectural state](https://en.wikipedia.org/wiki/Architectural_state)，却共享相同的[Execution unit](https://en.wikipedia.org/wiki/Execution_unit) 。后来出现的[多核处理器](https://en.wikipedia.org/wiki/Multi-core_processor)，却是在单个[芯片载体](https://en.wikipedia.org/wiki/Chip_carrier)上封装了多个物理/“真”的内核。

HT 能否带来性能提升取决于如下因素：

1. 操作系统的[调度器](https://en.wikipedia.org/wiki/Scheduling_(computing))是否知晓 CPU 的这一特性并能采取相应调度策略——举例来说，一台计算机安装了两个物理 CPU，都支持 HT 特性，但其操作系统内核（如线程调度器）却未对 HT 做相应的优化调整，进而认为 4 个逻辑内核都一样。如果当下有两个可以运行的线程，他们就可能被分配到同一物理 CPU 上，造成一个 CPU 特别忙，另外一个特别闲，但若被分配到不通的物理 CPU 的内核上，运行性能可能会更高；
2. 应用程序本身是否适用于这样的场景：一个逻辑内核会因为例如 [cache miss](https://en.wikipedia.org/wiki/CPU_cache#Cache_miss) 、[branch misprediction](https://en.wikipedia.org/wiki/Branch_misprediction) 、[data dependency](https://en.wikipedia.org/wiki/Data_dependency) 而进入 [CPU stalls](https://en.wikipedia.org/wiki/CPU_cache#CPU_stalls) 状态，另个逻辑内核就会利用空闲的执行资源执行自己的计算任务。
3. 程序员实现的代码也完美地利用了上述场景。

由于 HT 技术对应用的实现者要求比较高，潜在还会带来能耗和安全性方面地问题，现在已较少使用。

Wikipedia 拓展阅读

> - [Symmetric multiprocessing](https://en.wikipedia.org/wiki/Symmetric_multiprocessing)
> - [Parallel computing](https://en.wikipedia.org/wiki/Parallel_computing)
> - A [Microprocessor](https://en.wikipedia.org/wiki/Microprocessor) is a type of [Central processing unit](https://en.wikipedia.org/wiki/Central_processing_unit)
> - [Out-of-order execution](https://en.wikipedia.org/wiki/Out-of-order_execution)

## Tip

如何准备一个特定大小的文件？

通过 [`dd`](http://man7.org/linux/man-pages/man1/dd.1.html) 或 [`fallocate`](http://man7.org/linux/man-pages/man1/fallocate.1.html) 创建的文件都会被分配到“实在”的磁盘空间，区别是 `fallocate` 分配磁盘块不会涉及真正的 IO 操作，执行速度要比传统 `dd` （填满 null 字符）快上不少。

但上面两个命令都要注意：生成太大的文件可能超出用户的磁盘限额。但通过 [`truncate`](http://man7.org/linux/man-pages/man1/truncate.1.html) 创建的[稀疏文件](https://en.wikipedia.org/wiki/Sparse_file)可以是任何大小，不受磁盘限额的制约。

{{< highlight txt >}}
[foo@bar ~]$ dd if=/dev/zero of=anotherGBfile  bs=1G count=1
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB) copied, 2.73951 s, 392 MB/s
{{< /highlight >}}

{{< highlight txt >}}
[foo@bar ~]$ dd if=/dev/zero of=tenGBfile  bs=1G count=10
dm-1: warning, user block quota exceeded.
dm-1: write failed, user block limit reached.
dd: writing `tenGBfile': Disk quota exceeded
2+0 records in
1+0 records out
1763004416 bytes (1.8 GB) copied, 4.21954 s, 418 MB/s

[foo@bar ~]$ quota -v
Disk quotas for user foo (uid 505):
     Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
/dev/mapper/vg_foo_baz-lv_home
                6500004* 6000000 6500000   6days   41437       0       0
[foo@bar ~]$ rm -f tenGBfile
[foo@bar ~]$ fallocate -l 5G fiveGBfile
dm-1: warning, user block quota exceeded.
dm-1: write failed, user block limit reached.
fallocate: fiveGBfile: fallocate failed: Disk quota exceeded
[foo@bar ~]$ rm -f fiveGBfile

[foo@bar ~]$ quota -v
Disk quotas for user foo (uid 505):
     Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
/dev/mapper/vg_foo_baz-lv_home
                4778316  6000000 6500000           41436       0       0
[foo@bar ~]$ truncate -s 1T oneTBfile
{{< /highlight >}}

可用管理员权限使用 `repquota` 查看磁盘配额报告，并对某个用户的配额做出修改 `edquota`。
{{< highlight txt >}}
[foo@bar ~]$ sudo repquota -a
[sudo] password for foo:
*** Report for user quotas on device /dev/mapper/vg_foo_baz-lv_home
Block grace time: 7days; Inode grace time: 7days
                        Block limits                File limits
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      -- 7629792       0       0            111     0     0
qemu      -- 9002460       0       0              3     0     0
foo       -- 4778316 6000000 6500000          41436     0     0
......
[foo@bar ~]$ sudo edquota -u foo
{{< /highlight >}}

此外，如果期望文件是其他的形式：

- 内容非 null 的非可读文件：指定 dd 的 if 为 `/dev/urandom`
- 内容重复的文本文件：含 2^(n+1) 行的文件
- 内容随机的文本文件：安装 `yum install words`，以 `/usr/share/dict/words` 文件作为单词的输入源，并指定需要产生多少行，每行多少个单词，写个单行程序，详见 Alan Skorkin (2010-03-21) [How To Quickly Generate A Large File On The Command Line (With Linux)](https://skorks.com/2010/03/how-to-quickly-generate-a-large-file-on-the-command-line-with-linux/)

{{< highlight txt >}}
cat - > file.txt
hello
world
Ctrl+D to exit
for i in {1..n}; do cat file.txt file.txt > file2.txt && mv file2.txt file.txt; done
{{< /highlight >}}

拓展阅读：

> - Tom Hale answer @stackexchange [When to use /dev/random vs /dev/urandom](https://unix.stackexchange.com/a/324210)
> - Thomas Hühn (2019-02-17) [Myths about /dev/urandom](https://www.2uo.de/myths-about-urandom)

## Share

继续学习 Anghel Lenoard 的《Data Stream Development with Apache Spark, Kafka, and Spring Boot》

流式数据采集完为什么不直接接入分析层，而是通过消息队列传递(Kafka或RabbitMQ)呢？

- Backpressure issue
- Data Durability issue
- Data Delivery Semantics issue


封面图片来自 [D I Y](https://dribbble.com/shots/2862812-D-I-Y) <a href="https://dribbble.com/bryanfoo"><i class="fa fa-dribbble" aria-hidden="true"></i> Bryan Foo</a>