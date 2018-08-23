+++
date = "2018-08-10T22:28:13+08:00"
title = "ARTS 2018w32"
showonlyimage = false
image = "/img/blog/arts-06-per-week/cafe-hugo.jpg"
draft = false
weight = 1832
tags = ["ARTS", "Hugo"]
+++

终于动手修改了本博客的版式
<!--more-->

## Algorithm

本周 leetcode 题目 [167. Two Sum II - Input array is sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/description/) 和上周做过的 [1. Two Sum](https://leetcode.com/problems/two-sum/description/) 不同点在于：

- 给定数组已经做过升序排序
- 返回的索引要从 1 开始计数
- 返回的索引要保证先小后大的顺序

因为原题未要求必须使用[原地算法](https://en.wikipedia.org/wiki/In-place_algorithm)，依然借助 dict 作为缓存

{{< highlight python >}}

def twoSum2_cache(numbers, target):
    dic = {}
    for i, part in enumerate(numbers):
        prev_part = target - part
        if prev_part in dic:
            return [dic[prev_part]+1, i+1]
        dic[part] = i
{{< /highlight >}}

或不引入更多空间占用，通过“二分法”利用数组已排序的特性快速寻找潜在的匹配解:

{{< highlight python >}}
def twoSum2_binary_search(numbers, target):
    for i in range(len(numbers)):
        l, r = i+1, len(numbers)-1
        wanted = target - numbers[i]
        while l <= r:
            mid = l + (r-l)//2
            if numbers[mid] == wanted:
                return [i+1, mid+1]
            elif numbers[mid] < wanted:
                l = mid+1
            else:
                r = mid-1
{{< /highlight >}}

最后，利用“奇数和偶数之和一定为奇数”设定测试用例

{{< highlight python >}}
@pytest.fixture(params=[twoSum2_cache, twoSum2_binary_search])
def diff_solution(request):
    return request.param


def test_1_even_with_many_odd(diff_solution):
    chk_iter = 1000
    for _ in range(chk_iter):
        len_a = randint(2, chk_iter)
        even_i = randint(0, len_a-1)

        input_a = [2*i-1 for i in range(0, even_i)]
        input_a.append(2*even_i)
        input_a += [2*i-1 for i in range(even_i+1, len_a)]

        odd_j = randint(0, len_a-1)
        while odd_j == even_i:
            odd_j = randint(0, len_a-1)

        expect_i = [even_i+1, odd_j+1]
        expect_i.sort()

        m, n = diff_solution(input_a, input_a[even_i] + input_a[odd_j])
        assert [m, n] == expect_i
{{< /highlight >}}

TODO: 采集不同算法的性能数据，并用[可视化工具查看](https://rakyll.org/pprof-ui/)

## Review 

本周看了 Giraffe Academy 录制的 [Hugo - Static Site Generator Tutorial](https://www.youtube.com/playlist?list=PLLAZ4kZ9dFpOnyRlyS-liKL5ReHDcj4G3) 算是对 gohugo 有一个大概的了解：


{{< figure src="/img/blog/arts-06-per-week/hugo.jpg" title="Hugo Sketchnote" >}}

更改的版式包括：

- 去除单篇文章的左边栏，使用全屏独占模式
- 为文章加入标签，点击可以查看带有相同标签的所有文章

## Tip

Gerrit 是网页版的代码审查工具，但也可以安装一个 python 工具 git-review 通过命令行做相应操作。

可以跟随 Tutorialpoint 上的 [Gerrit Tutorial](https://www.tutorialspoint.com/gerrit/index.htm) 做安装使用的学习。

## Share

本月的一则新闻提到 Google 可能正在尝试开发一个能进入我国的定制版的搜索引擎。众多回应中比较醒目的有：人民日报表示欢迎但提醒要遵守我国法律。而李彦宏则表示百度有信心再赢一次。

而从外媒也我看到两篇报道：

- 纽约时报（2018-08-16）[Google Employees Protest Secret Work on Censored Search Engine for China](https://www.nytimes.com/2018/08/16/technology/google-employees-protest-search-censored-china.html)
- 华盛顿邮报 (2018-08-16) [Google’s censored search engine could actually help Chinese citizens](https://www.washingtonpost.com/opinions/dont-be-so-quick-to-write-off-googles-censored-search-engine-for-china/2018/08/16/208a518e-a09a-11e8-b562-1db4209bd992_story.html?noredirect=on&utm_term=.5a3962bcae70)

前一篇文章说谷歌员工希望自己的公司公开透明自然没问题，担心自己的工作成果被不当使用，而且感觉这些员工谁都怼，对谷歌参与美国国防部项目也同样表示抗议。

而后一篇文章我得承认是被标题吸引过去的，我的观点是查询技术资料方面，谷歌的搜索结果非常靠谱。只要纯技术部分能进来，应该对我们的科技发展大有脾益。而看到这篇的标题，心想我的想法居然和外媒暗合，这什么情况啊 ? 结果点进去一看，作者所谓的“对中国人民有益”还是基于意识形态、民主的考量，整篇短文散出一股股悲天悯人的救世主情怀……

各人的思路真的很不一样。

## 拓展阅读

> - yihui Xie [Hugo theme Xmag](https://github.com/yihui/hugo-xmag/)


封面图片来自 [Cafe Hugo](https://dribbble.com/shots/1755553-Cafe-Hugo) <a href="https://dribbble.com/SC_Letterpress"><i class="fa fa-dribbble" aria-hidden="true"></i> Sacrés Caractères</a>