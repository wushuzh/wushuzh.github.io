+++
date = "2019-02-18T16:41:16+08:00"
title = "ARTS 2019w08"
showonlyimage = false
image = "/img/blog/arts-35-per-week/matrix.webp"
topImage = "/img/blog/arts-35-per-week/matrix.gif"
draft = false
weight = 1908
tags = ["ARTS","linux","memory","kubernetes", "unionfind"]
+++

linux 内存入门及一个 swap 优化案例
<!--more-->

## Algorithm

[leetcode #128 Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) ：给定任意一维数组，求其元素可能组成的最长递增1的数列长度。

如依然通过 Union Find 数据结构求解。思考的重点放在如何将给定数组向 union find 数据结构映射：

- Union-Find 内部数组 parent 的索引**必须**和给定数组一致，用于唯一标识各个节点，而索引指代的值**必须**为父节点的索引；
- 为了求解最大递增序列的长度，在 Union-Find 内部再引入一个数组 nodesNum，表示各个节点下挂接的子节点数量(这个属性后续可以用于数据结构扁平化)；
- 为了更方便的检测连续，需要引入一个 dict ：key 为给定数组的各个元素值，这样就可以快速判定比当前元素大1的元素是否存在，value 为给定数组的索引；

特别注意：

- 合并两个不同的 root 时，新选出的 root 的 nodesNum 要更新为之前两个 root 各自已有子节点数量之和
- python 将 0 视为 false，考虑到 -1 + 1 = 0 ，判断条件要严格限定为`dict.get(any int) is not None`，而不是简单的判定`dict.get(any int)` 为 true

{{< highlight python "linenos=inline, hl_lines=8" >}}
def longest_consecutive_by_union_find(nums: List[int]) -> int:
    if not nums:
        return 0
    
    uf = UnionFind(nums)
    valToIdx = {nums[i]: i for i in range(len(nums))}
    for v in valToIdx.keys():
        if valToIdx.get(v + 1) is not None:
            uf.union(valToIdx[v], valToIdx[v+1])
    return uf.largest_one_union()
{{< /highlight >}}


## Review

本周学习了 [Linux Under the Hood](https://learning.oreilly.com/live-training/courses/linux-under-the-hood/0636920257462) 内存相关的章节

从上层应用角度看，[管理内存](https://en.wikipedia.org/wiki/Memory_management) 的对象其实是 [Virtual memory](https://en.wikipedia.org/wiki/Virtual_memory)。 

从操作系统角度看：  
- [管理内存](https://en.wikipedia.org/wiki/Memory_management_(operating_systems)) 的主要对象是 [Primary memory（如内存）](https://en.wikipedia.org/wiki/Computer_data_storage#Primary_storage)；  
- 额外地，为了能高效利用 [Secondary storage（如硬盘）](https://en.wikipedia.org/wiki/Computer_data_storage#Secondary_storage) 支持大型程序的运行，引入了 [Paging（分页）](https://en.wikipedia.org/wiki/Paging)  
- 硬盘的容量大，但随机读取速度慢，为了完成更高效查找而引入的 [Page table（分页表）](https://en.wikipedia.org/wiki/Page_table)、[TLB（页表缓存）](https://en.wikipedia.org/wiki/Translation_lookaside_buffer) 和 [Page cache](https://en.wikipedia.org/wiki/Page_cache) ——一系列以 [Page](https://en.wikipedia.org/wiki/Page_(computer_memory)) 的核心的高效操作；

```
                USER
                 ^
                 |
                 |
                 |
          +------v------+               +--------+
          |        PAGE CACHE           |        |
          |   PROC DATA |<--------------+        |
          |   PAGE TBL  |     PAGING    |        |
PROCESS +-+     (TLB)   |               |  HDD   |
          |             +-------------->|        |
          |       DIRTY CACHE           |        |
          +------^------+               +--------+
              RAM|
                 |
             +---v----+
             |     |L1|
             |     +--+
             | CPU |L2|
             |     +--+
             |     |L3|
             +--------+
```

拓展阅读：  

> - Michael Boelen (2016-10-31, last updated 2018-7-3) [Understanding memory information on Linux systems](https://linux-audit.com/understanding-memory-information-on-linux-systems/)
> - Mel Gorman free book [Understanding the Linux Virtual Memory Manager](https://www.kernel.org/doc/gorman/)
> - torvalds/linux @github [ T H E  /proc   F I L E S Y S T E M](https://github.com/torvalds/linux/blob/master/Documentation/filesystems/proc.txt)
> - Redhat Knowledge base [Interpreting /proc/meminfo and free output for Red Hat Enterprise Linux 5, 6 and 7](https://access.redhat.com/solutions/406773)
> - Robert Love answer @ Quora [What is the difference between Buffers and Cached columns in /proc/meminfo output?](https://qr.ae/TWT9eO)
> - Doug Breaker (2015-4-10) [Understanding page faults and memory swap-in/outs: when should you worry?](https://scoutapm.com/blog/understanding-page-faults-and-memory-swap-in-outs-when-should-you-worry)
> - Thomas-Krenn wiki [Linux Page Cache Basics](https://www.thomas-krenn.com/en/wiki/Linux_Page_Cache_Basics)

课程描述了一个内存优化的实例：先用 `free -m` 查看内存整体情况：16 GB 物理内存，1 GB SWAP，SWAP  100 %使用；进而通过`cat /proc/meminfo |grep -i active` 查看内存整体占用中其中 anno 内存（用于进程）活跃内存的仅 1GB，不活跃的居然白白占用了 14 GB空间；

{{< highlight txt "hl_lines=5">}}
cat /proc/meminfo |grep -i active
Active:           2 GB
Inactive:        15 GB
Active(anon):     1 GB  # process request
Inactive(anon):  14 GB  #   shall be put into swap and free for other usage
Active(file):   0.5 GB  # cache/buffer
Inactive(file):   0 GB
{{< /highlight >}}

解决方案就是增加 SWAP 内存：从 1G 升到 16 GB，并同时调整 swappiness 参数 `echo 90 > /proc/sys/vm/swappiness`，然操作系统更激进的将 anno inactive 内存放入 SWAP；立刻查看 `vmstat 3` 观察 swap
 in/out 数值，后续运行几天后，总体呈现了一个健康的内存分配情况：

{{< highlight txt "hl_lines=5">}}
cat /proc/meminfo |grep -i active
Active:           4 GB
Inactive:         9 GB
Active(anon):     1 GB  # process request
Inactive(anon):   1 GB  #   is more eager to swap in
Active(file):     3 GB  # cache/buffer
Inactive(file):   8 GB
{{< /highlight >}}

**因此，一些确定swap大小的简单规则：** 

- 通用服务器起始值，50% 物理内存（<4GB)，25 %（>4GB）；
- 服务器稳定运行后 inactive anno 内存的量，应该等于 swap 大小；
- 特殊大型应用，比如 Oracle ，swap 大小直接设为物理内存的 1.5 倍；
- 需要使用休眠功能的笔记本，swap 大小至少设为和物理内存一样；

增加 swap 的相关指令：

- `fdisk /dev/sdx` 创建 partition type 为 82 （swap）的分区；
- `mkswap /dev/sdx1` 初始化 swap 文件系统；
- `swapon /dev/sdx1` 开启 swap 分区；
- `free -m` 或 `swapon -s` 查看 swap 空间；
- `/etc/fstab` 按 `fs-mntpoint-type-options-dump-pass` 格式添加新行 `/dev/sdx1  none  swap   sw  0   0`

## Tip

top 可以用来查看系统中进程列表，支持的操作有

- 展开查看在每个 CPU core 上的负载，按 `1`；
- 杀掉进程，按 `k` 输入进程号；
- 按内存、CPU等其他列排序，`M`（内存）、`P`（CPU）……；
- 更通用的方法，`f` 进入显示配置页面，上下选择感兴趣的列，`d`显示开关，`s`为排序；
- 切换线程视图，按`H`来回切换；
- 显示完整进程命令行，按`c`；
- 切换为父子关系的进程视图，按`V`；
- 仅列出某个用户的进程，按`u`,输入用户名；
- 高级过滤，按'O',输入表达式，`COMMAND=getty` 或 `%CPU>10.0`

详见 [A Guide to the Linux “Top” Command](https://www.booleanworld.com/guide-linux-top-command/) 以及 [top manpage](http://man7.org/linux/man-pages/man1/top.1.html)

## Share

Brendan Burns et al., "Borg, Omega, and Kubernetes: Lessons Learned from Three Container-Management Systems over a Decade", ACM Queue 14 (2016): 70–93.

文章是 Google 在开发了 Borg, Omega, k8s 后的总结，我快速读完，列出一些我认为的要点：

- 容器技术达到了“质变”：容器使得依赖和应用运行环境管理，标记(anno)，资源分配和限制、负载均衡等等都是面向“应用”的（和现实世界映射更容易），比如资源分配和性能监控，完全不用先从主机角度收集，之后再分离了到各应用上。同时 POD 代表的一个应用实例，如果应用比较复杂，完全可以拆分不同功能到多个容器中，既保证了紧凑工作，又可以分别同步开发。
- k8s 所有 API 设计都非常一致：内容由 Objmetadata、Spec(desired state)、Status(read-only) 组成。API 不但用于从外部访问（无论是普通用户或是外部工具），而且 k8s 的控制面内部也都是通过相同的 API 实现所有功能；
- k8s 的 API 功能足够分离，一些高阶功能直接依赖底层功能，比如 Horizontal Pod Autoscaler 仅负责应用需求收集和预测，决定是否加减实例数目，而 ReplicationController 来保证实例数量；再比如 ReplicationController、DaemonSet、Job 内部都依赖于 Pod ，在 Pod 之上针对不同需求各自实现不同特性；
- 各种对象的状态靠独立观察，没有内部状态留存，这样管理组件遇到问题，直接重启也没有问题；

关于什么不要做的教训：

- 不要试图管理容器的端口：不然，ip+port的方式和 dns url 都不一致，各种转换会变成“灾难”；
- 不要试图对容器按数字命名，要用标签：不然，处理死掉的实例、多集群协同、对于分片(sharding)应用的升级都不好处理；而标签管理则强大的多：动态增删改，分组可重叠，对于 statefulset 可以同时使用 idx 和 label；
- 要小心设计“从属关系”：单一的父子关系看似简单，但只要设计合理，基于标签的松耦合可能更为灵活(svc 和 rs 一起管理)；
- 不要暴露原始状态：对于开源系统来说，暴露太多的原始信息就意味着会和某个具体实现发生关联，k8s 通过中控 API server 来做内部验证，对外保持高度一致性，屏蔽不必要细节，使得组件可替换；

关于那些 Google 觉得也还没解决好的问题：应用配置、依赖管理（启动配置所依赖的服务）

封面图片来自 [Zest Matrix](https://dribbble.com/shots/6206009-Zest-Matrix) <a href="https://dribbble.com/johnwlong"><i class="fa fa-dribbble" aria-hidden="true"></i> John W. Long</a>