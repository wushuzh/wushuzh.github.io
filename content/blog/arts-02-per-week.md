+++
date = "2018-07-10T22:59:17+08:00"
title = "ARTS 2018w28"
showonlyimage = false
image = "/img/blog/arts-02-per-week/missinglink.png"
draft = false
weight = 1828
tags = ["ARTS", "bigdata", "tools"]
+++

如何模拟数据、以及大数据处理
<!--more-->

## Algorithm

本周解析两道 leetcode 试题: [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/) 和 [80. Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/description/) 

放在一起是因为解题思路完全一致，还是双指针法: 快指针`j`"勇往直前"，轮询所有元素，慢指针`i`标记当前符合题设要求的最新元素索引位置，通过两个指针标记的元素们的关系，判断`j`的位置上是否是一个新的符合题设的元素，若是，就将`i`增加 1 ，将新元素拷贝到索引`i`上。

{{< highlight python >}}
# 026_rm_dup_from_sorted_array
def rm_duplicates(nums: List[int]) -> int:
    if len(nums) <= 1:
        return len(nums)
    i = 0
    j = 1
    while j < len(nums):
        if nums[j] > nums[i]:
            i += 1
            nums[i] = nums[j]
        j += 1
    return i + 1
{{< /highlight >}}

{{< highlight python >}}
# 080_rm_dup_from_sorted_arr_2  
def rm_dups(nums: List[int]) -> int:
    if len(nums) <= 2:
        return len(nums)
    i = 1
    j = 2
    while j < len(nums):
        if nums[j] > nums[i] or nums[j] > nums[i - 1]:
            i += 1
            nums[i] = nums[j]
        j += 1
    return i + 1
{{< /highlight >}}

leetcode 的题目关注算法和运行效率，代码微调不可避免，所以最好为算法建立测试集。

但除了用于测试边界的用例外，测试集还需要覆盖哪些情况才算完备？我的做法是通过程序自动生成符合题设规则数据，原则上这要生成的数组足够多，就能覆盖所有情况。

比如，下面用例就是重复进行1000 次测试，每次生成一个 100 个以内、依次递增的不同元素组成的数组`a`的测试函数。

{{< highlight python >}}
# test_solution.py for 026_rm_dup_from_sorted_array
def test_multi(random_chk=1000):
    for _ in range(random_chk):
        a = []
        no_dup = randint(2, 100)
        for i in range(no_dup):
            repeat = randint(1, 5)
            [a.append(i) for _ in range(repeat)]
        assert rm_duplicates(a) == no_dup
        assert a[:no_dup] == list(range(no_dup))
    # TODO: get the branchmark of a function
{{< /highlight >}}

## Review 

本周学习 Kafka 时看到几篇讨论其设计的博客都引用了一张比较硬盘和内存在随机和连续模式下读取数据速度的图示。

<img alt="Random vs Sequential" src="/img/blog/arts-02-per-week/comparsion.png" class="img-responsive">

图上写着 Figure 3 , 那么其他图又会是什么？原文为何做了这种比较，具体上下文是什么？好奇驱使去查了一下图片原始来源。

原文是刊登在 ACM Queue 杂志 2009-07 Volume 7 Issue 6 的一篇文章 [The Pathologies of Big Data](https://queue.acm.org/detail.cfm?id=1563874) 作者是来自 [1010data Inc.](https://www.1010data.com/company/our-story/) 的 [Dr. Adam Jacobs](https://www.1010data.com/company/leadership/dr-adam-jacobs/)。网上搜了一下，1010data 的最重要客户是纽交所，对于大数据分析应该是很有一套，而 Adam Jacobs 现在已是公司的首席科学家。

回顾历史，Google 在 2003 年发布了著名的关于其文件系统以及 MapReduce 的论文。三年后，也就是 2006 年，业界基本将论文消化完成，发布了 Hadoop 项目 1.0 版本。在经过 3 年，工业界有了一些大数据应用。

设想你如果此时希望写一篇关于存储和处理大数据的科普文章，该如何切入话题呢？

Adam 采用了实验实证的方式。

他首先提到了美国 1980 年的人口普查，全部数据达到了 100 GB ，这些记录被存放在 IBM MSS(Mass Storage System) 系统内，每 GB 的存储成本高达 4 万美元——当时看来硕大无比的数据，30年后再看呢？容量 1T 的硬盘，电脑城俯拾皆是，100 美元就能带回家。再说数据处理，作者模拟了一次全世界人口普查，每人占用一条大小为 128 bits 的记录，全球按 60~70 亿的人口数量计算，刚好是一个 100 GB 大小的数据集:

- 7 bits 记录年龄 (最大 127 岁)
- 1 bit 存放性别
- 8 bits 标识国家 (现下全世界 195 个国家)
- ...

计算目标设定为找出各国男女人口各自的年龄中值。采用最平实的算法，瓶颈是硬盘读取速度，运行 15 分钟可以得到结果——期间 CPU 基本赋闲，紧接着 Adam 又将全部数据导入一台刚好拥有 128 GB 内存的 Dell 服务器中，同样的算法，不到一分钟就能得到了结果。

<img alt="Simulation" src="/img/blog/arts-02-per-week/fake_data.png" class="img-responsive">

在作者看来，能这样处理的数据根本算不上大数据。但公平的地说，这种规模的数据用常规关系型数据库处理已经非常有挑战了。为了证明这一观点，作者又准备了另一个实验：找一台苹果工作站: 20 GB 内存，2 TB 硬盘，将这 6~7 十亿条数据导入 PostgreSQL 数据库，首次尝试以失败告终，因为持续 6 个小时的导入将硬盘写满了，然而数据还没有全导完。第二次只好将仅含国家、年龄、性别三个字段的 10 亿条记录导入库中，这相当于原来 2 % 的数据量，然而 DBMS 存储也花费了 40 GB硬盘——这叫做“数据通胀” ^_^

而下图中这条 SQL 语句的执行花费了 24 小时。作者在使用 EXPLAIN 工具分析了这个聚合操作的执行计划后，解释了其算法在处理十亿量级的数据的局限，说明了传统数据库一大问题在于其设计目标本来就是用于高效地执行小批量数据的事务型操作的，对于海量的、整体累计型的聚合计算则显示出了严重的不适应。 

<img alt="linear cost" src="/img/blog/arts-02-per-week/query-cost.png" class="img-responsive">

之后作者提出了大数据在时间、空间上**连续性**的本质，并花费大量篇幅说明只有在存储和算法上都遵从、顺应这一本质的处理策略才有前途。要考虑不同介质的存储成本、空间大小、级联传输效率等因素的影响，尤其要考虑如何利用好如前图 3中不同介质在随机和连续两种模式下时读取速度的指数级的差异。

之后作者又分析了分布式计算的应用策略和需要注意的问题。最后阐述了他对于大数据内涵的元定义。整篇文章绝对是科普佳品，在我看来即使放在今日，其可读性和知识性都很高。建议大家找来阅读。

## Tip

[tmux](https://en.wikipedia.org/wiki/Tmux) 是 linux 的命令行神器。(然而我还没有完全用起来)

而 Windows 下一个类似得命令行工具也很值得推荐: [cmder](http://cmder.net/) 它深度整合了 Conemu 、clink、git for windows 三个工具于一身: 提供了可更换的主题，命令行修改回查的增强，使得 Windows 命令行爱好者也可以获得类似 tmux 的体验?

对我来说常用的操作有:

- 像 Quake 一样用热键随时唤起 cmder
- 自定义热键(如双 Ctrl)复制一个当前 shell ，平铺于当前面板右侧
- 向在当前面板的所有 shell 中同时发送指令

> 默认的热键定义大量使用了 [Apps Key](https://conemu.github.io/en/AppsKey.html) 通常在全尺寸键盘的 RWin 和 RCtrl 键之间，但对于 Thinkpad 这样的笔记本键盘上这个键并不存在，需要借助 Sharpkey 这样的工具将另一个不常用的键映射为 Apps Key 才可使用，详见 [Lenovo W530 and the Context Menu Key](https://blogs.msdn.microsoft.com/timid/2013/09/19/lenovo-w530-and-the-context-menu-key/)

## Share

本周点评文章中提到了美国人口普查，我发现它还真是计算机发展的一个重要现实推手。

在[PBS 的计算机科学速成课](https://www.youtube.com/playlist?list=PL8dPuuaLjXtNlUrzyH5r6jN9ulIgZBpdo) 中第二集就提到美国人口普查办公室委托 [Herman Hollerith](https://en.wikipedia.org/wiki/Herman_Hollerith) 实现的打孔卡片制表机(现在计算机的雏形)使得 1890 年的人口普查工作得以在 1 年内完成。(相比 1880 年却花了 8 年)

经常听到人们夸耀美国真最牛啊:自由、民主、三权分立，政治制度设计完备。但有没有可能这些其实都是表象，甚至是噱头。这背后的本质是它拥有比其他政府强大的多的管理能力。信息技术才是它的核心硬实力——保持在多维、海量数据的收集和处理能力上的绝对领先，然后再在这之上再构建什么几乎都能成功。所以说无论是对组织，对个人，拥有理工科思维，能尝试用概率统计的角度去理解世界真的非常重要。

最近看了不少贸易战、中兴事件的报道，也挺有意思: 如果你是班级里第一名，不但自己觉得风光，大家也都很羡慕。第一名能力大，大家也愿意也赋予其更多的职责权力，提出的建议大家也都附和，而且当了这么多年一哥也慢慢摸清了里面的门道，行走江湖基本能做到名利双收，不太会赔本赚吆喝。然而这么风光的位置，其他人也想上啊，尤其是第二名，但这么多年过去了，仍然是没有一个能真正威胁到一哥。最新的竞争者是一副东方面孔，据说特点是“勤劳勇敢”，嗯，之前也有过另一位来自东方的同学竞选过一哥：爱干净，懂礼貌，但性格有点阴郁……

{{< figure src="/img/blog/arts-02-per-week/father.jpg" title="罗立中 《父亲》" >}}

封面图片来自 [The Missing Link](https://dribbble.com/shots/4008681-The-Missing-Link) <a href="https://dribbble.com/BenStafford"><i class="fa fa-dribbble" aria-hidden="true"></i> Ben Stafford</a>