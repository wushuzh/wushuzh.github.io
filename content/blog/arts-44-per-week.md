+++
date = "2019-04-22T10:41:57+08:00"
title = "ARTS 2019w17"
showonlyimage = false
image = "/img/blog/arts-44-per-week/dns.webp"
topImage = "/img/blog/arts-44-per-week/dns.gif"
draft = true
weight = 1917
tags = ["ARTS"]
+++

待定
<!--more-->

## Algorithm

[leetcode #928 Minimize Malware Spread II](https://leetcode.com/problems/minimize-malware-spread-ii/) 此题和 [leetcode #924 Minimize Malware Spread](https://leetcode.com/problems/minimize-malware-spread/) 表述基本一致——都是在某些病毒源节点中，找出去除后能最小化网络病毒扩散的一个节点。不同点在于，此题去除时比较彻底，明确要求将 graph 中有关联的链接也一并去除——原来 N * N 矩阵变为 (N - 1) * (N - 1) 矩阵。

Union Find 数据结构中并没有删除连接关系的操作，更不要说是删除一系列的连接关系。所以本题一个重要的思考角度是，将网络节点分为病毒源节点和非病毒源节点。然后在此基础上进行统计分析，从而评估这些连接关系消失后对网络的影响。

1. 先将所有非病毒源节点放入 union find 中完成连接；
2. 逐一求得每个病毒源节点对上述网络中产生影响的独立 group 的列表——用每个 group 的 root 节点作为 group_id；
3. 当这些受影响的 group 中只有一个病毒源时，清除此病毒源可以“拯救”整个group；
4. 轮询每个病毒源，找出影响最大的返回即可；

{{< highlight python "linenos=inline, hl_lines=15">}}
def min_malware_spread_II_using_unionfind(graph: List[List[int]],
                                          initial: List[int]) -> int:
    N = len(graph)
    clean = set(list(range(N))) - set(initial)

    uf = UnionFind(N)

    for i in clean:
        for j in clean:
            if graph[i][j]:
                uf.union(i, j)

    # global view from virus
    #  one virus can impact which groups: id by root of a group
    impact_clean_nodes = collections.defaultdict(set)
    for v in initial:
        for i in clean:
            if graph[v][i]:
                impact_clean_nodes[v].add(uf.find_root(i))

    # global view from clean nodes
    #  one clean node can be impacted by how many virus
    impact_clean_counts = collections.Counter()
    for v in initial:
        for i in impact_clean_nodes[v]:
            impact_clean_counts[i] += 1

    max_impact_num = -1
    ans = min(initial)
    # loop with each virus to see
    #   how many groups it can independently impact
    for v, some_clean_nodes in impact_clean_nodes.items():
        cnt = 0
        for i in some_clean_nodes:
            if impact_clean_counts[i] == 1:
                cnt += uf.sz[i]

        if cnt > max_impact_num or (cnt == max_impact_num and v < ans):
            max_impact_num = cnt
            ans = v

    return ans
{{< /highlight >}}

## Review

拓展阅读：

> - Justin Ellingwood and Mitchell Anicas from digitalocean [An Introduction to Managing DNS](https://www.digitalocean.com/community/tutorial_series/an-introduction-to-managing-dns)
> - 鸟哥的 Linux 私房菜 [主机名控制者：DNS 服务器](http://cn.linux.vbird.org/linux_server/0350dns_1.php)

## Tip

[#22 Selecting columns with visual block mode](http://vimcasts.org/episodes/selecting-columns-with-visual-block-mode/)

[vim ctrl V -> ctrl Q not working](https://github.com/cmderdev/cmder/issues/1686)

[tmux copy paste 模式]

## Share






封面图片来自 [1.1.1.1](https://dribbble.com/shots/4441857-1-1-1-1) <a href="https://dribbble.com/karilinder"><i class="fa fa-dribbble" aria-hidden="true"></i> Kari Linder</a>