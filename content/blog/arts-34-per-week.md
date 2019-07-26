+++
date = "2019-02-11T19:03:13+08:00"
title = "ARTS 2019w07"
showonlyimage = false
image = "/img/blog/arts-34-per-week/"
topImage = "/img/blog/arts-34-per-week/"
draft = true
weight = 1907
tags = ["ARTS"]
+++

待定
<!--more-->

## Algorithm

[leetcode #200 Number of Islands](https://leetcode.com/problems/number-of-islands/) 是计算一个“二维地图”中联通的块数量（上下、左右的值均为1即视作联通)。

其中一种解法是借助一种数据结构完成的 —— Coursera 课程 [Algorithms, Part I](https://www.coursera.org/learn/algorithms-part1) 第一周介绍的数据结构：[并查集](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) 

- 数据结构的定义：用数组元素定义任一节点: 1) 数组元素的 idx 保证唯一可作为每个节点的唯一标识 2) 此索引下对应的值表示和祖先节点的唯一标识 idx —— 显然，若为孤岛元素此值与自己的 idx 相同
- 基本操作包括 1) 找出一个给定节点的“祖先”节点 **func root(i)**，从而判断任意两个节点是否有链接关系 **func is_connected(p, q)** 2) 将任意两个节点连接到一起 **union(p, q)**

拆解如下：

1. 基于题目给定输入 List[List[str] 定义和初始化 UnionFind 并查集的数据结构；
2. 实现两个基本操作 find_root 和 union ；
3. 基于上面的数据结构，判定相邻元素，调用 union ，当全部轮询结束时，就得到了结果；
4. 按照 coursera 课程提示进行优化：扁平化数据结构，提升 union 操作效率

{{< highlight python >}}
def find_num_islands_with_uf(grid: List[List[str]]) -> int:
    if not grid:
        return 0

    uf = UnionFind(grid)
    r, c = len(grid), len(grid[0])
    for i in range(r):
        for j in range(c):
            if grid[i][j] == '1':
                # check its right neighbor till right bound
                if j + 1 < c and grid[i][j + 1] == '1':
                    uf.union(i * c + j, i * c + j + 1)
                # check its down neighbor till down bound
                if i + 1 < r and grid[i + 1][j] == '1':
                    uf.union(i * c + j, c * (i + 1) + j)
    return uf.count
{{< /highlight >}}

## Review  

Stéphane Maarek 的 Kafka 系列课程 Kafka Stream 学习总结：

{{< figure src="/img/blog/arts-34-per-week/kafka-stream.jpg" title="Kafka Stream" >}}

## Tip

Golang 声明主要两种形式，但都能明确确定下来变量的类型：

- 正式的 `var varName anyType` 分配内存后，值被初始化为 **zero value**；
- 内部的 `var varName := inferableVal` 只有典型值，但编译器可以从值推断出类型；

以 struct 类型为例，定义一个复合型数据结构 struct 语法：

```
struct T structName {
    fieldName fieldType
    // other fields with type per line
}
```

struct literal



## Share

封面图片来自 []() <a href=""><i class="fa fa-dribbble" aria-hidden="true"></i> </a>