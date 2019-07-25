+++
date = "2019-03-11T21:41:39+08:00"
title = "ARTS 2019w11"
showonlyimage = false
image = "/img/blog/arts-38-per-week/security.webp"
draft = false
weight = 1911
tags = ["ARTS","Kafka"]
+++

KAFKA 加密、认证、权限的相关知识总结
<!--more-->

## Algorithm

[leetcode #547 Friend Circles](https://leetcode.com/problems/friend-circles/)：将 n 个人放在[对称矩阵](https://en.wikipedia.org/wiki/Symmetric_matrix) 表示互相的朋友关系，求最后朋友圈的数量。

本题用 Union-Find 数据结构解决，和之前看到的并查集的矩阵相关问题不同，本题的关键点在于：

- n 个人作为 Union-Find 中的节点，而不是矩阵中的每个元素；
- 独立朋友圈的数量，应该是最终 n 个节点中“祖先”节点的数量——是否能有一种可通过矩阵直观辨识出答案的方法？；
- 对称矩阵中有一半信息都是冗余的，注意不要重复处理；

{{< highlight python "linenos=inline, hl_lines=5 9" >}}
def find_circle_num_using_union_find(M: List[List[int]]) -> int:
    uf = UnionFind(len(M))
    for i in range(uf.num):
        # ignore upper-right triangle in matrix
        for j in range(0, i):
            if M[i][j] == 1:
                uf.union(i, j)

    return len([1 for i, v in enumerate(uf.parent) if i == v])
{{< /highlight >}}

## Review

尝试着不看教程，读官网总结相关知识：

{{< figure src="/img/blog/arts-38-per-week/Proemetheus.jpg" title="Proemetheus" >}}

## Tip

Golang 中的 println 和 fmt.Println:

虽然前者不需要任何依赖，但其输出是 stderr ，只应该用在开发者的调试的场景；
fmt包内的各种 Print* 和 Fprint* 输出为 stdout ，在生产环境中使用；

以 Golang 的值传递为例：

{{< highlight go >}}
package main

func main() {
        count := 1
        println("count:\tvalue Of[", count, "]\tAddr Of[", &count, "]")
        increment(count)
        println("count:\tvalue Of[", count, "]\tAddr Of[", &count, "]")
}

func increment(inc int) {
    inc++
    println("inc:\tvalue Of[", inc, "]\tAddr Of[", &inc, "]")
}
{{< /highlight >}}

{{< highlight txt >}}
count:	value Of[ 1 ]	Addr Of[ 0x41a7ac ]
inc:	value Of[ 2 ]	Addr Of[ 0x41a7a8 ]
count:	value Of[ 1 ]	Addr Of[ 0x41a7ac ]
{{< /highlight >}}

- 每个 Goroutine 会分配一个相对较小(2KB)的其专属 stack，而传统每个线程 stack 大小为 1MB；
- 任何函数调用都会从 stack 中分配一个新 frame，专门存储此函数的相关数据——这个新 frame 是 Goroutine 当前唯一能直接访问的内存；
- 数据都有其值(what)和其地址(where），Golang 中 函数传参都采用值传递；

## Share

Stéphane Maarek 的 Kafka 系列课程 Kafka Security (SSL SASL Kerberos ACL) 学习总结：

{{< figure src="/img/blog/arts-38-per-week/kafka-security.jpg" title="Kafka Security" >}}


封面图片来自 [Web security](https://dribbble.com/shots/3603008-Web-security) <a href="https://dribbble.com/Faannette"><i class="fa fa-dribbble" aria-hidden="true"></i> Fanny Algeyer</a>