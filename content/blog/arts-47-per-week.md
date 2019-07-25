+++
date = "2019-05-13T13:59:44+08:00"
title = "ARTS 2019w20"
showonlyimage = false
topImage = "/img/blog/arts-47-per-week/"
draft = true
weight = 1920
tags = ["ARTS"]
+++

开始学习 GCP 
<!--more-->

## Algorithm

[leetcode #162 Find Peak Element](https://leetcode.com/problems/find-peak-element/) 用来查找局部峰值，注意 leetcode #162 要返回峰值元素的索引，而不是元素本身。另一个完全相同的题目是 [leetcode #852 Peak Index in a Mountain Array](https://leetcode.com/problems/peak-index-in-a-mountain-array/)

{{< highlight python >}}
def find_peak_element_recursive(nums):
    def search(nums, l, r):
        if l == r:
            return l
        m = (l + r) // 2
        if nums[m] < nums[m + 1]:
            return search(nums, m + 1, r)
        return search(nums, l, m)

    search(nums, 0, len(nums) - 1)
{{< /highlight >}}

{{< highlight python >}}
def find_peak_element_iterative(nums):
    l = 0
    r = len(nums) - 1
    while l < r:
        m = (l + r) // 2
        if nums[m] > nums[m+1]:
            r = m
        else:
            l = m + 1

    return l
{{< /highlight >}}

MIT 6.006 第一课首先讲的就是此题，进一步还讲解如何查找 2 维矩阵中的局部峰值：即找出这样的元素，比它上、下，左，右的相邻元素都大。解决方案都是：[二分搜索](https://en.wikipedia.org/wiki/Binary_search_algorithm) + [分治法](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm)。

计算机科学中，需要[分析算法](https://en.wikipedia.org/wiki/Analysis_of_algorithms)的运行（时间）效率 : 特别是处理海量输入时，算法执行用时将如何随之增长？这里引入了 [Big O notation](https://en.wikipedia.org/wiki/Big_O_notation) 或 asymptotic notation 。

对于一个被评估函数 g(n)，若存在一个可用于表示上限和下限的（最简单/化形式）渐进函数，将 g(n) 曲线完全包裹。

- 当上下限函数非常相像（接近）仅因子不同时（如 n^2 和 1.2n^2），则使用 [大写希腊字母 Theta](https://en.wikipedia.org/wiki/Theta#Upper_case) 表示为 Θ(n^2)
- 当上下限函数不同：不仅因子不同，肩头指数等也不同时，则其下限表示为 Ω(f(x)) ，上限用 O(f(x)) 表示

算法的效率分析通常关注上限函数，所以日常使用中常常忽略 Θ 和 Ω ，而只采用大写 O 的记法。

拓展阅读 [MIT 6.006 Introduction to Algorithms, Fall 2011](https://www.youtube.com/playlist?list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)

> - Instructor: Srini Devadas [1. Algorithmic Thinking, Peak Finding](https://youtu.be/HtSuA80QTyo)
> - Instructor: Victor Costan [R1. Asymptotic Complexity, Peak Finding](https://youtu.be/P7frcB_-g4w)
> - Minghan 2018-09-25 [Golden Section Search — Peak Index in a Mountain Array](https://medium.com/datadriveninvestor/golden-section-search-method-peak-index-in-a-mountain-array-leetcode-852-a00f53ed4076)

## Review


## Tip



## Share



封面图片来自 []() <a href=""><i class="fa fa-dribbble" aria-hidden="true"></i> </a>