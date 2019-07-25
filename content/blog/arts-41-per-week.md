+++
date = "2019-04-01T23:37:10+08:00"
title = "ARTS 2019w14"
showonlyimage = false
image = "/img/blog/arts-41-per-week/hexagon.webp"
topImage = "/img/blog/arts-41-per-week/hexagon.gif"
draft = false
weight = 1914
tags = ["ARTS"]
+++

强烈推荐李永乐老师的科普视频
<!--more-->

## Algorithm

[leetcode #959 Regions Cut By Slashes](https://leetcode.com/problems/regions-cut-by-slashes/) 在 N*N 正方形区域中挑选一部分小格，并选择两条对角线中的一条划实，求解最终图形中不连通的区域数量。

通过题目标签可知可以用 Union Find 解决，像任何一个用并查集算法的问题一样，关键在于如何将问题(中的图形)转化为 Union Find 数据结构。

- 将所有小格分为上、下、左、右 4 个三角形，将这些三角形作为基本的合并单元；
- 所有垂直、水平相邻两个三角形都一定要 union；
- 含对角线的小格内部，三角形两两做 union；
- 完全不含对角线的小格内部，4 个三角形互相 union；
- 上面最后的两个情况，可以考虑合并，简化代码； 

{{< highlight python "linenos=inline,hl_lines=10 14-23" >}}

def regions_by_slashes_using_union_find(grid: List[str]) -> int:
    if not grid:
        return 0

    uf = UnionFind(grid)
    N = len(grid)

    for i, row in enumerate(grid):
        for j, slash in enumerate(row):
            ijIdx = 4 * (i * N + j)
            #    1
            # 0     3
            #    2
            if slash == '/':
                uf.union(ijIdx + 0, ijIdx + 1)
                uf.union(ijIdx + 2, ijIdx + 3)
            elif slash == '\\':
                uf.union(ijIdx + 0, ijIdx + 2)
                uf.union(ijIdx + 1, ijIdx + 3)
            else:
                uf.union(ijIdx, ijIdx + 1)
                uf.union(ijIdx, ijIdx + 2)
                uf.union(ijIdx, ijIdx + 3)

            # union down 1/4 cell except last row N - 1
            if i < N - 1:
                uf.union(ijIdx + 2, ijIdx + 4 * N + 1)

            # union up 1/4 cell except first row 0
            if i >= 1:
                uf.union(ijIdx + 1, ijIdx - 4 * N + 2)

            # union right 1/4 cell except first column N - 1
            if j < N - 1:
                uf.union(ijIdx + 3, ijIdx + 4)

            # union left 1/4 cell except last column 0
            if j >= 1:
                uf.union(ijIdx + 0, ijIdx - 1)

    return sum(uf.find_root(i) == i for i in range(4 * N * N))
{{< /highlight >}}

## Review

学习 Anghel Lenoard 的《Data Stream Development with Apache Spark, Kafka, and Spring Boot》

**流式数据接入**的几种方式：

- Webhooks
- HTTP Long Polling
- Server Sent Events
- WebSocket

{{< figure src="/img/blog/arts-41-per-week/ingestion.jpg" title="Streaming data collection" >}}

## Tip

最近碰到一个和 cron 相关的问题，在服务器上安装启动了 [Performance Co-Pilot](https://pcp.io/docs/pcpintro.html): pcp日常运行中会在 `/var/log/pcp/pmlogger/<host>` 下生成一些存档文件，而在 `/etc/cron.d/` 下也配置了相应的周期性清理。

但是我们服务器上使用的 pcp 清理脚本存在问题：在一些特殊情况下，其处理的半拉子日志文件能造成 purge 脚本运行时提前退出，无法正常完成清理工作，造成最后日志分区爆满。Redhat 确认了在我们使用的 pcp 版本中的问题([pmlogger_daily is unable to discard or compress older files which failed compression for some reason](https://bugzilla.redhat.com/show_bug.cgi?id=1697182))，提供了一个新版本 pcp-4.3.2-2，然而只在 RHEL8 中提供，RHEL7 还要等一段时间。好在这个问题有方法绕过，即手动清理掉半拉子日志文件，之后的清理脚本就可以正常工作。

{{< highlight txt >}}
::::::::::::::
/etc/cron.d/pcp-pmlogger
::::::::::::::
# daily processing of archive logs (with compression enabled)
10     0  *  *  *  pcp  /usr/libexec/pcp/bin/pmlogger_daily -X xz -x 3
{{< /highlight >}}

{{< figure src="https://imgs.xkcd.com/comics/cron_mail.png" title="XKCD #1728: CRON MAIL" >}}

## Share

李永乐老师的科普视频 [石墨烯是什么？](https://youtu.be/adIN_RjIrmY)

{{< figure src="/img/blog/arts-41-per-week/carbon.jpg" title="Graphene" >}}

李老师的科普视频真的好棒，强烈推荐。


封面图片来自 [Hexagonal Hard Candy Revisited - 20](https://dribbble.com/shots/2851969-Hexagonal-Hard-Candy-Revisited-20) <a href="https://dribbble.com/AdmiralPotato"><i class="fa fa-dribbble" aria-hidden="true"></i> Admiral Potato</a>