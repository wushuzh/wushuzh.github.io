+++
date = "2018-07-01T22:13:07+08:00"
title = "ARTS 2018w27"
showonlyimage = false
image = "/img/blog/arts-01-per-week/infinite-trip.png"
topImage =  "/img/blog/arts-01-per-week/infinite-trip.gif"
draft = false
weight = 1827
+++

关于宇宙意志、文明等级的假说
<!--more-->

本周申请了左耳朵耗子老师专栏的读者ARTS活动，即每周完成一个 ARTS 

- **A**lgorithm: 一个 leetcode 算法题
- **R**eview: 点评一篇英文技术文章
- **T**ip: 学习一个技术技巧
- **S**hare: 分享一个技术观点和思考

## Algorithm

这是注册 leetcode 后的第一题，直接挑了一道简单的：原地算法将给定数组中特定值清除。 
题目详情见 [27 Remove Element](https://leetcode.com/problems/remove-element/description/)

首先一个无视规则的天真做法是这样的，事实上这里的 for 语句后面对数组的 slice 操作是新建了一个数组，不符合题设对于[原地算法](https://en.wikipedia.org/wiki/In-place_algorithm)的要求。

{{< highlight python >}}
def deprecated_rm_elem(nums: List[int], val: int) -> int:
    for i in nums[:]:
        if i == val:
            nums.remove(val)
    return len(nums)
{{< /highlight >}}
<br />

双指针法：创建两个从0开始的数组索引，遇到要删除的元素，仅向前移动快指针，否则拷贝快指针指向的元素到慢指针位置，然后同时向前移动两个指针。

{{< highlight python >}}
def rm_elem_2_points(nums: List[int], val: int) -> int:
    i = j = 0
    while j < len(nums):
        if nums[j] != val:
            nums[i] = nums[j]
            i += 1
        j += 1
    return i
{{< /highlight >}}
<br />

因为上述方法遇到保留的元素需要拷贝，在最差情况两个指针都要 2n 步，所以在欲删除的元素极少出现的情况下，可以设定两个指针分别从头尾向中间移动，遇到需要删除的元素，就从数组后面替换一个过来，再检测换来的元素是否需要再换，直到两指针相遇。

{{< highlight python >}}
def rm_elem_swap(nums: List[int], val: int) -> int:
    i = 0
    n = len(nums)
    while i < n:
        if (nums[i] == val):
            nums[i] = nums[n - 1]
            n -= 1
        else:
            i += 1
    return n
{{< /highlight >}}
<br />

## Review 和 Tip

看到阮一峰老师发的亚马逊工作法，去原 blog 读了下，十多年前的帖子了。只是简单总价了一下，没做啥点评。 

不过最近正在看 BDD ，觉得背后的理念有点像，就顺带总结了一下 cucumber 的核心概念，就算 Tip 
详见 [测试驱动开发]({{< relref "blog/head-first-cucumber.md" >}})

## Share

本来应该分享技术观点，但我觉得下面这个也行吧？

这周读了薛定谔的《生命是什么》的第一章，提到和原子相比，为什么我们的身体那么长？极简的解答是只有参与生命体的分子达到一定数量，才足以抵消单体运动产生的扰动和误差，从而使得生物体本身亦能更明显地展现，同时感知到“白噪音”之外的那些更加深刻的主宰宇宙的规律和趋势。

我借题发散了一下，如果将上述论断迁移到人类社会，即众多个体相对松散地组成了社群、公司、国家等各种组织。一方面，我们都不得不做着一些常规运动，另一方面，个体是否能理解比它自身尺度大得多地组织背后的运行规律？进而对所在组织地当前和未来走势做出感知和预测？那些所谓牛人是否就是能某种程度上能超越自身无意义的运动，有能力去感知和践行将更大尺度上的规律，并展示给其他个体和整个组织的人？

<img alt="Cosmos" src="/img/blog/arts-01-per-week/cosmos.gif" class="img-responsive">
[Hawking](https://dribbble.com/shots/4363039-Hawking) <a href="https://dribbble.com/merittthomas"><i class="fa fa-dribbble" aria-hidden="true"></i> Meritt Thomas</a>

说到假说，罗胖最近推荐一本书叫《生命 3.0》，提到一个观点：如果说宇宙有意志的话，那么这个意志可能是熵增。物理和天文学家推断宇宙的一个可能结局是走向热寂，这也是宇宙“意志”的最终体现：即将万事万物扩散成一种无聊而又完美的匀质的状态。而这个约束无比之强大，以至于只要是在本宇宙中的任何动作都只能加重这一趋势: 你收拾房间，房间变得更有序了，但你因此花费的能量会使得房间之外的世界更加无序。

> 据说这和薛定谔《生命是什么》中论述的观点一致：生命系统的一个标志就是，通过增加周围环境的熵，来降低自己的熵——显然我还没有读到这一章节。

然后生命之所以"被创造"出来，就是因为它更耗能，产生无序的效率更高，进而人类之所以成为地球主宰，也就是因为我们能产生比其他生命体高出几个数量级的能量消耗。(抬一下杠，坐禅、苦修这类行为对自身的自律其实也是更耗费能量的)无论如何，宇宙总是更加偏爱和鼓励更耗能的行为。

想起一次我在天文馆听的讲座，提到了一种文明分类的方式:

- 能充分利用所在行星上能量的文明是一星文明——比如将地球大地印刷为电路板
- 能100%利用所属恒星能量的文明可称为二星文明——比如将太阳用塞森球套住
- 能自由利用所在星系能量的文明为三星文明——没法想象，阿西莫夫的银河帝国都不算吧？

当下我们人类处于 0.7 星文明。而按照上述宇宙意志的假说，我们不断追求文明的过程也是更快消耗能量，加速熵增，引领我们走向热寂的过程。

<img alt="Red Planet" src="/img/blog/arts-01-per-week/planet.gif" class="img-responsive">
[HELLO DRIBBBLE](https://dribbble.com/shots/3994298-HELLO-DRIBBBLE) <a href="https://dribbble.com/funktional"><i class="fa fa-dribbble" aria-hidden="true"></i> Funktional</a>

封面图片来自 [Infinite trip](https://dribbble.com/shots/2900680-Infinite-trip) <a href="https://dribbble.com/Grei"><i class="fa fa-dribbble" aria-hidden="true"></i> Grei</a>