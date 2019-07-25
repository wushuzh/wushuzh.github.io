+++
date = "2019-03-04T20:10:31+08:00"
title = "ARTS 2019w10"
showonlyimage = false
image = "/img/blog/arts-37-per-week/equation.webp"
topImage = "/img/blog/arts-37-per-week/equation.gif"
draft = false
weight = 1910
tags = ["ARTS","linux","grab", "ingestion"]
+++

如何给定条件，做更多的推导判断
<!--more-->

## Algorithm

[leetcode #399 Evaluate Division](https://leetcode.com/problems/evaluate-division/)：除法计算，给定一些除法等式和其商值，即 `a / b` 可知商为 k （强制要求 k 为正实数）：

一个具体的例子：`a / b` 值为 2.0 ，`b / c` 值为 3.0， 如何才能导出如下算式的值：

- `a / c` 为 6.0
- `b / a` 为 0.5
- `a / a` 为 1
- `a / e` 无解，
- `x / x` 无解

解题思路：Union-Find ，equations 视为 union 操作，若 query 的两个变量有连接到同一祖先节点，则有解——解为各自到祖先 factor 相除。

注意：

- 表示两个节点的除法关系时，认为下面两种表示法等价 `a / b -> k` `b / a -> 1 / k`；
- union 操作不能覆盖已有连接关系；(TODO: 当下实现将 x 和 y 做挂接，应该再考虑对 xRoot 和 yRoot 挂接)
- 为求解未知节点间的倍数关系，引入了一个附加数据结构，表示当前节点到 root 的商，因此 find 操作在递归查找 root 时要顺带更新此数值；

{{< highlight python "linenos=inline, hl_lines=14-16，23-29" >}}
class UnionFind():
    def __init__(self, equations):
        self.parent = dict()
        self.factor = dict()
        for i in set([x for e in equations for x in e]):
            self.parent[i] = i
            self.factor[i] = 1.0

    def find_root(self, i):
        if self.parent.get(i) is None:
            return None
        if self.parent[i] != i:
            # accumulate_factor case
            to_be_replaced_parent = self.parent[i]
            self.parent[i] = self.find_root(self.parent[i])
            self.factor[i] *= self.factor[to_be_replaced_parent]
        return self.parent[i]

    def union(self, x, y, v):
        xRoot = self.find_root(x)
        yRoot = self.find_root(y)
        if xRoot != yRoot:
            # TODO: 当下实现将 x 和 y 做挂接
            #   应该再考虑对 xRoot 和 yRoot 挂接，也许能进一步简化问题
            if yRoot == y:
                self.parent[y] = x
                self.factor[y] = 1 / v
            else:
                self.parent[x] = x
                self.factor[x] = v

def calc_equation_by_union_find(equations: List[List[str]],
                                values: List[float],
                                queries: List[List[str]]) -> List[float]:

    uf = UnionFind(equations)
    for i, e in enumerate(equations):
        numer, deno = e
        factor = values[i]
        uf.union(numer, deno, factor)

    ans = []
    for x, y in queries:
        xRoot = uf.find_root(x)
        yRoot = uf.find_root(y)
        # new variable not exist in equations, result must -1.0
        if None in (xRoot, yRoot):
            ans.append(-1.0)
        else:
            if xRoot != yRoot:
                ans.append(-1.0)
            else:
                ans.append(uf.factor[x] / uf.factor[y])

    return ans
{{< /highlight >}}

## Review

[How we simplified our Data Ingestion & Transformation Process](https://engineering.grab.com/data-ingestion-transformation-product-insights)

早听说 Grab 的主力语言是 Go，本篇文章介绍了 Grab 的流式实时数据采集系统：

1. 开始原型搭建，尽量采用开源流式处理方案 SparkStreaming，开发快速，并帮助理解数据转换层的处理实质；
2. 发现开源软件限制（python protobuf解码器）和 S3 最终一致性的限制，导致延迟和数据丢失难于修复；
3. 用 Go 重新实现 SparkStreaming 的所有处理功能，最终解决了延迟，新老方案结果比较验证重构逻辑正确；

{{< figure src="/img/blog/arts-37-per-week/grab-ingestion.jpg" title="Grab ingestion subsystem" >}}


## Tip

在 Linux 下，任意一个用户进程，其默认中等优先级（20），不冷不热的友好（0）。

- 使用 `renice` 命令更改进程的 nice 值，进而变更进程的优先级。nice 范围 [-20, +19] 从“很不友好”，到“老好人”——和现实世界有点像的是：表现的越不 nice ，获得的优先级越高；
- 通过 nice [-20, 19] 可以影响的是普通进程，优先级范围是 [0, 39]；
- 还有一些被标记为 `rt` （实时）进程，优先级范围是从 [-100, -1]；
- `chrt` 可以直接的改进程的优先级；

## Share

[`/proc` 目录下又都有啥？](https://www.thegeekdiary.com/understanding-the-proc-file-system/)

{{< figure src="/img/blog/arts-37-per-week/proc.jpg" title="Understand '/proc'" >}}

封面图片来自 [I△ + △▽ + [◐ - D] = AND](https://dribbble.com/shots/1686039-I-D-AND) <a href="https://dribbble.com/justinlawes"><i class="fa fa-dribbble" aria-hidden="true"></i> Justin Lawes</a>