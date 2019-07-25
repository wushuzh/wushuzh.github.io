+++
date = "2019-04-29T19:09:37+08:00"
title = "ARTS 2019w18"
showonlyimage = false
image = "/img/blog/arts-45-per-week/grid.webp"
topImage = "/img/blog/arts-45-per-week/grid.gif"
draft = true
weight = 1918
tags = ["ARTS"]
+++

待定
<!--more-->

## Algorithm

[leetcode #839 Similar String Groups](https://leetcode.com/problems/similar-string-groups/) 给定一些由同样字母组成的单词，只需将两个不同位置的字母交换一次的得到的两个单词定义为相似，将给定的单词列表按照上述相似条件分组，求最终有多少不同的组。

这道题是典型的 Union Find 问题，若两个单词相似，就将其 union 起来。

{{< highlight python >}}
def is_similiar(w1, w2):
    cnt = 0
    for i1, i2 in zip(w1, w2):
        cnt += (i1 != i2)
    return cnt in (0, 2)
{{< /highlight >}}

但解法提交后，总是标识超时。主要是在两两比较单词（双层嵌套loop)太过耗时，选出两个单词，然后逐一对比字符串。当单词量特别大时，应该尝试转换为单层循环，然后主动将其中的两个字母交换顺序，然后查看是否需要 union。这样性能至少可以被 leetcode 提交系统接受。

{{< highlight python >}}
def num_similiar_groups_using_union_find(A: List[str]) -> int:
    def is_similiar(w1, w2):
        cnt = 0
        for i1, i2 in zip(w1, w2):
            cnt += (i1 != i2)
            if cnt > 2:
                return False
        return cnt == 2

    uf = UnionFind(A)

    strlen, wordlen = len(A[0]), len(A)
    if wordlen < strlen * strlen:
        for i, w1 in enumerate(A[:-1]):
            for w2 in A[i + 1:]:
                if is_similiar(w1, w2):
                    uf.union(w1, w2)
    else:
        given_words = set(A)
        for w in given_words:
            for i in range(strlen):
                for j in range(i + 1, strlen):
                    new = w[:i] + w[j] + w[i + 1:j] + w[i] + w[j + 1:]
                    if new in given_words:
                        uf.union(w, new)

    return len({uf.find_root(w) for w in uf.p})
{{< /highlight >}}

## Review



## Tip


## Share

[Gluster](https://en.wikipedia.org/wiki/Gluster) arch

https://docs.gluster.org/en/latest/Quick-Start-Guide/Architecture/

Performance https://youtu.be/h86zGibyndM

internal hash algo http://www.taocloudx.com/index.php?a=shows&catid=4&id=66

heketi

https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.4/html/administration_guide/ch05s02



封面图片来自 [Breath](https://dribbble.com/shots/4060778-Breath) <a href="https://dribbble.com/BestServedBold"><i class="fa fa-dribbble" aria-hidden="true"></i> 𝔅𝔢𝔰𝔱𝔖𝔢𝔯𝔳𝔢𝔡𝔅𝔬𝔩𝔡</a>