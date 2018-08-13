+++
date = "2018-07-23T22:47:27+08:00"
title = "ARTS 2018w30"
showonlyimage = false
image = "/img/blog/arts-04-per-week/rule1.png"
draft = false
weight = 1830
tags = ["ARTS", "sketchnote", "redhat"]
+++

Sketchnote 更有效的笔记记录方式？
<!--more-->

## Algorithm

本周解析的题目是 [82. Remove Duplicates from Sorted List Ⅱ](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/description/)

和上周 83 题相比，唯一的区别是对重复元素的处理——这次要求彻底清除。但两题解法完全不同

1. 引入一个 dummy ListNode 作为头部，如此才能删除输入链表的任意元素
2. 推动两个指针 p(prev) q(curr) 前移的过程中，通过变更 next 进行重复元素删除

{{< highlight python >}}
def deleteDuplicates(head: ListNode) -> ListNode:
    dummy = ListNode(0)
    dummy.next = head
    p, q = dummy, dummy.next

    while p and q:
        found_duplicate = False
        while q.next and q.next.val == q.val:
            found_duplicate = True
            p.next = q.next
            q = q.next

        if found_duplicate:
            p.next = q.next
        else:
            p = p.next

        q = q.next

    return dummy.next
{{< /highlight >}}

测试数据的生成函数和上次基本相同。但以后要考虑如何将公用函数提出来，供相同数据结构的题目使用。

{{< highlight python >}}
def test_multi(check_itr=1000):
    for _ in range(check_itr):
        in_a = []
        expect_a = []
        last_num = randint(2, 100)
        for i in range(last_num):
            repeat = randint(1, 3)
            if repeat == 1:
                expect_a += [i]
            in_a += [i] * repeat
        new_head = deleteDuplicates(mklist(*in_a))
        assert convert_array(new_head) == expect_a
{{< /highlight >}}


## Review 

看到一篇文章，介绍一位外国妹子[Tess](https://twitter.com/TessFerrandez)学习[吴恩达的机器学习课程](https://www.coursera.org/instructor/andrewng)后，做的一系列[笔记](https://www.slideshare.net/TessFerrandez/notes-from-coursera-deep-learning-courses-by-andrew-ng)。

原来这种笔记叫做 sketchnote ，在此领域比较出名布道者大概算是 Mike Rohde ? 他出过书，繁体翻译版的书名为“[直覺式塗鴉筆記](https://www.jianshu.com/p/ca5cf7102c94)”， Mike Rohde 在 Youtube 上视频: 

> [Sketchnote Mini Workshop - Interaction South America 2017](https://youtu.be/39Xq4tSQ31A)

正在 Peachpit 上看视频课程 The Advanced techniques for taking visual notes you can use anywhere , 觉得这种方式笔记方式非常有意思。但这些视频居然没有倍速播放的设置，其他操作也不太好用。

{{< figure src="/img/blog/arts-04-per-week/sketchnote.jpg" title="Sketchnote from Chris Spalton" >}}

## Tip

去年的这个时候连续碰到了几次操作维护的问题，然后弄了一个“咋整”系列。

前几天同事在一次系统大升级时遇到问题，然后试图重启服务器就一直在进度条处。我当时没时间参与分析，提了一些建议，然后在事后交流了一下。 因为公司购买了 Redhat 技术支持，这种问题不怕解决不了，总有 Redhat 兜底。

0. 启动服务器，注意观察自检期间的硬件报错/异常
1. 如果 GRUB 有问题，就通过一个烧有安装介质的U盘或光盘启动，比如 [How to Recover or Rescue Corrupted Grub Boot Loader in CentOS 7](https://www.tecmint.com/recover-or-rescue-corrupted-grub-boot-loader-in-centos-7/)
2. 如果能进入 GRUB2 启动菜单，首先可以去除 rhgb 和 quiet 属性，并设置日志级别后继续启动，比如 [CentOS / RHEL 7 : How to change the verbosity of debug logs during booting](https://www.thegeekdiary.com/centos-rhel-7-how-to-change-the-verbosity-of-debug-logs-during-booting/)
3. 发现了问题点后，可以尝试进入救援模式对系统进行修复，比如 [CentOS / RHEL 7 : How to boot into Rescue Mode or Emergency Mode](https://www.thegeekdiary.com/centos-rhel-7-how-to-boot-into-rescue-mode-or-emergency-mode/)
4. 对于 Redhat 的服务器，救援模式下它的 sosreport 工具直接可用，可以一键收集系统所有的日志，详见[How to generate sosreport from the rescue environment](https://access.redhat.com/solutions/2872)
5. 收集完日志就可以启动网络，将日志传出来，详见 [Enabling networking in rescue environment without chrooting](https://access.redhat.com/solutions/2626631)
6. 之后就需要在本地研读各种日志找出具体的问题了，这部分最有技术含量，根据各人对系统启动整个流程的熟悉程度，找问题的时长可能差出几个数量级

最后发现问题是一个错误设置的系统参数，Redhat Solution 也早有答案 [Getting lots of "kernel: VFS: file-max limit XXXXX reached" messages in logs, how do I fix it?](https://access.redhat.com/solutions/24725) 初步分析找到了一个脚本，在系统在非正常的状态下，本身实现中缺少防御保护，算出了一个极其不合理的值并写入了 sysctl.conf 中，然后机器就不能启动了。

## Share

Mike Rohde 说他曾经记过很多很多传统意义上的笔记：记得越快，忘得越快，而且是记完后就扔到了一边，再也没有看过。

我对这句话印象很深刻，可能是因为这样的经历在我身上也重演过很多遍，为什么会造成这种结果？

1. 学习的内容不在自己当下的技术路径上，虽说任何知识都“有用”，但若不能经过**实践**、**总结**、**重学**形成一个可反馈的闭环，就会使当初投入的时间、精力迅速贬值；
2. 冗长单调的文字叙述使人麻木，其实不仅限于 sketchnote 还有思维导图、手动实验、复述交流等，都能重新激活思考，提升学习效率；
3. 单页的涂鸦笔记其实是“浓缩”后的知识，强制自己找出内部关联，形成高阶视角，是一个把书读薄的过程——若是能对着这页笔记滔滔不绝讲上两个钟头，就完成了把书再读厚；

行动指南：

- 在随身笔记本上，对看到、听到的一切内容练习这种涂鸦笔记;
- 关注效率提升，看视频听音频可以选择倍速播放，多尝试浓缩知识，节省他人和自己的时间；
- 不要放任自己去追技术热点，成体系的知识更有价值——按照陈皓老师的说法，一步步按攻略来，两三年必有小成。

封面图片来自 [Sketchnotebasics](https://dribbble.com/shots/2513889-Sketchnotebasics) <a href="https://dribbble.com/ChrisSpalton"><i class="fa fa-dribbble" aria-hidden="true"></i> Chris Spalton</a>