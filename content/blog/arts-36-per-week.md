+++
date = "2019-02-25T14:49:01+08:00"
title = "ARTS 2019w09"
showonlyimage = false
image = "/img/blog/arts-36-per-week/ml.webp"
draft = false
weight = 1909
tags = ["ARTS"]
+++

分布式系统：存储设计三种形式
<!--more-->

## Algorithm

[leetcode #130 Surrounded Regions](https://leetcode.com/problems/surrounded-regions/)：要求将图中可以被 **X** 全部包围的“内部” **O** 区域做反转，有点类似围棋中“吃子”的操作。

若是使用 Union-Find 来解决，思路是将边缘的 'O' 都链接到一个虚拟的额外节点上，之后就是常规的操作：将二维数组放入 UF 数据结构中，最后可以判断所有未链接到额外节点的 **O** 都是可以“被吃”的内部区域，做相应反转即可。

需要注意的是：

- 第 14，17，20 行的判断条件不可级联，都需要独立判断，边缘的节点和 dummy 节点链接后，也需要查看其右，其下是否有其他可联通的节点，为了防止索引溢出，第 17、20 行的条件要增强；
- 并查集并不保证 union 操作时的挂接顺序，不能写成 `if uf.find_root(i * uf.c + j) != uf.dummyIdx`, 这本质上是 Union-Find 的 is_connected 操作；
- 应该对 Union-Find 数据结构做扁平化，以提高性能；


{{< highlight python "linenos=inline, hl_lines=14，17，20，25" >}}
def flip_inside_O_by_UF(board: List[List[str]]) -> None:
    if not board:
        board = []
        return

    uf = UnionFind(board)
    dummyIdx = uf.r * uf.c

    for i in range(uf.r):
        for j in range(uf.c):
            if board[i][j] == 'O':
                ijIdx = i * uf.c + j
                # connect all boundary nodes to the ONE dummy node
                if i in (0, uf.r - 1) or j in (0, uf.c - 1):
                    uf.union(ijIdx, dummyIdx)
                # check its right neighbor is connected ?
                if j + 1 < uf.c and board[i][j + 1] == 'O':
                    uf.union(ijIdx, ijIdx + 1)
                # check its down neighbor is connected ?
                if i + 1 < uf.r and board[i + 1][j] == 'O':
                    uf.union(ijIdx, ijIdx + uf.c)

    for i in range(uf.r):
        for j in range(uf.c):
            if uf.find_root(i * uf.c + j) != uf.find_root(uf.dummyIdx):
                board[i][j] = 'X'
{{< /highlight >}}

## Review

来自 Cassandra CTO Tim Berglund 的关于分布式系统的快速扫盲课程 《Distributed Systems in One Lesson》。

关于分布式系统的存储，有三种方式：

- Read Replication
- Sharding
- Consistency Hashing

{{< figure src="/img/blog/arts-36-per-week/distributed-system-storage.jpg" title="Distributed System: Storage" >}}

## Tip

将某进程固定在某个 CPU core 上，防止进程在不同的 CPU core 间来回做切换。

{{< highlight txt >}}
dd if=/dev/zero of=/dev/null &

# enter top utility
#   `f` then add colume P: Last Used Cpu (SMP)
top

# pin dd process to cpu 3
#  taskset -pc [CORE-LIST] [PID]
taskset -pc 3 $(pidof dd)

# bring cpu 3 offline
echo 0 > /sys/bus/cpu/devices/cpu3/online
# double confirm with lscpu
lscpu

# enter top again to confirm dd is using another cpu core
top
{{< /highlight >}}

## Share

看了一下 GitOps 原理部分，还需要继续看一下具体实践是什么样的？

[Tutorial: Hands-on Gitops - Brice Fernandes, Weaveworks](https://youtu.be/0SFTaAuOzsI)

{{< figure src="/img/blog/arts-36-per-week/GitOps.jpg" title="GitOps Principles" >}}

扩展阅读：

WeaveWorks 2018-08-21 [What Is GitOps Really?](https://www.weave.works/blog/what-is-gitops-really)

封面图片来自 [Machine Learning](https://dribbble.com/shots/4806833-Machine-Learning) <a href="https://dribbble.com/igorkozak"><i class="fa fa-dribbble" aria-hidden="true"></i> Igor Kozak</a>