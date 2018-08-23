+++
date = "2018-08-15T19:33:04+08:00"
title = "ARTS 2018w33"
showonlyimage = false
image = "/img/blog/arts-07-per-week/satellite.jpg"
draft = false
weight = 1833
tags = ["ARTS", "linux", "tools"]
+++

利用参考实现进行测试，以关于比较优势的思考
<!--more-->

## Algorithm

本周 leetcode 题目 [2. Add Two Numbers](https://leetcode.com/problems/add-two-numbers/description/)，本质上这道题目就是实现一个正整数的加法。

{{< highlight python >}}
def add_two_numbers_v1(l1: ListNode, l2: ListNode) -> ListNode:
    carry = 0
    dummy = n = ListNode('dummy')
    while l1 or l2 or carry:
        v1 = v2 = 0
        if l1:
            v1 = l1.val
            l1 = l1.next
        if l2:
            v2 = l2.val
            l2 = l2.next
        carry, val = divmod(v1 + v2 + carry, 10)
        n.next = ListNode(val)
        n = n.next
    return dummy.next
{{< /highlight >}}

下面是 leetcode 用户 [StefanPochmann 的解法]( https://leetcode.com/problems/add-two-numbers/discuss/1102/Python-for-the-win) 它的算法可以接受任意多个链表作为输入。

其中比较有趣的部分:

1. 第2、7行的 `a=b=c` 作为一种 Chained assignment 其执行顺序是怎样的？
2. 第4、5、8行的 `afewlls` 的用法值得注意: 和[Comprehensions相关](https://python-3-patterns-idioms-test.readthedocs.io/en/latest/Comprehensions.html)

{{< highlight python "linenos=inline,hl_lines=2 4 5 7 8" >}}
def add_two_numbers_v2(*afewlls):
    dummy = n = ListNode("dummy")
    carry = 0
    while afewlls or carry:
        carry += sum(a.val for a in afewlls)
        carry, val = divmod(carry, 10)
        n.next = n = ListNode(val)
        afewlls = [a.next for a in afewlls if a.next]
    return dummy.next
{{< /highlight >}}

TODO: 

- StackOverflow 上用户 pfctdayelise 的自问[自答](https://stackoverflow.com/a/11498861/4393386): 维基百科上有[说明解释](https://en.wikipedia.org/wiki/Assignment_%28computer_science%29#Chained_assignment)，或阅读 Python 官方参考文档中[赋值语句符号](https://docs.python.org/reference/simple_stmts.html#assignment-statements) 但要先看一下[BNF grammar notation](https://docs.python.org/3/reference/introduction.html#notation)
- StackOverflow 上另一个答案 (https://stackoverflow.com/a/36346517/4393386) 利用 Python 标准模块 dis: [Disassembler for Python bytecode](https://docs.python.org/3/library/dis.html) 解释该问题


测试方面使用了 hypothesis 产生测试数据，并引入两个辅助函数实现 ListNode 和 正整数间的相互转换。这样我们就可以通过常规内置的 int 加法来验证 ListNode 加法。

{{< highlight python >}}
def rll2int(l):
    s = ''
    while l:
        s = str(l.val) + s
        l = l.next
    return int(s)


@ht.settings(verbosity=ht.Verbosity.verbose)
@ht.given(n1=st.integers(min_value=0), n2=st.integers(min_value=0))
def test_random(n1, n2, diff_solution):
    l1 = mklist(*map(int, reversed(str(n1))))
    l2 = mklist(*map(int, reversed(str(n2))))
    r = diff_solution(l1, l2)
    assert rll2int(r) == n1 + n2
{{< /highlight >}}

总结起来，这种测试思路，可以理解为另外完成一个原题目的参考实现：即如果两种解法的实现思路非常不同，那么针对任意一个测试用例，这两种实现同时犯错的概率是很低的。比如下面这种解法很不适宜作为面试题的解答，但作为一个参考实现是可以的。

{{< highlight python >}}
def add_two_numbers_v3(l1, l2):
    def toint(node):
        return node.val + 10 * toint(node.next) if node else 0
    def tolist(n):
        node = ListNode(n % 10)
        if n > 9:
            node.next = tolist(n / 10)
        return node
    return tolist(toint(l1) + toint(l2))
{{< /highlight >}}

## Review 

读了一下 Uber 博客 (2018-04-19) Danny Iland, Andrew Irish, Upamanyu Madhow and Brian Sandler [Rethinking GPS: Engineering Next-Gen Location at Uber](https://eng.uber.com/rethinking-gps/)

{{< figure src="/img/blog/arts-07-per-week/gps.jpg" title="Uber GPS Sketchnote" >}}

> - (2015-9-7) [Urban Localization and 3D Mapping Using GNSS Shadows](http://insidegnss.com/urban-localization-and-3d-mapping-using-gnss-shadows/)
> - (2018-04-23) [Uber Marketplace: Fixing GPS in Cities](https://youtu.be/-6JZSV3jjLA)

## Tip

三个月前就激活了 Windows 10 的 WSL (Windows Subsystem for Linux) 从应用商店安装 Debian ，但直到这周看推，看到阮一峰老师推荐的终端录制工具 termtosvg ，细读之下发现[暂不支持 Windows](https://github.com/nbedos/termtosvg/issues/13) 才下定决心进入 Debian 试一下。

1. 若习惯使用 Cmder ，可以为其[自定义一个新任务 bash::debian](https://gingter.org/2016/11/16/running-windows-10-ubuntu-bash-in-cmder/)
2. 听说 Debian 区别于 Ubuntu 的一个重要不同是默认不安装 sudo ，但 WSL 上的 Debian 已预装 sudo: [`apt list --installed`](https://www.cyberciti.biz/faq/how-to-list-all-installed-packages-debian-ubuntu/)
3. 若无法直连外网，要先[设置 apt 代理](https://wiki.debian.org/AptConf) ，然后执行 `sudo apt update` 但后续仍可能遇到[代理缓存过期引起的 Hash Sum Mismatch 问题](https://askubuntu.com/a/709568)

{{< highlight bash >}}
sudo apt install python3 python3-pip

pip3 install --user pipenv
# https://github.com/pypa/pipenv/issues/2122#issuecomment-386207878
echo 'export PATH="${HOME}/.local/bin:$PATH"' >> ~/.profile
pip3 install --user termtosvg
{{< /highlight >}}

下面是使用 termtosvg 录制的执行本周 leetcode 测试的操作:

<img style="width:70%; height:70%; display:block; margin: auto 10%;" alt="debian in windows" src="/img/blog/arts-07-per-week/wsl-debian.svg" class="img-responsive">

## Share

关于卫星导航系统，看了知乎用户 Moien 转载的我国工程院院士，许其凤教授在2012年7月27日，天津滨海国际会展中心举办的“战略性新兴产业（滨海）国际论坛的[讲演](https://www.zhihu.com/question/21092045/answer/92719534)。

其中提到80年代，以东芝、夏普为代表的日本彩电品牌优势已经非常明显了，我们是否还需要发展自己的彩电产业？许院士有一个观点：即便不能做到在整体上实现全面超越，但如果可以形成某方面的相对超越也存在抢回市场的可能。日本彩电由于普遍离发射塔距离比较近，基于此前提设计的接收机灵敏度在我国就表现得“差强人意”，而国产彩电对此做出针对性的改进就能使得“雪花”比日产的更少。另外销售、服务本地化我们也可以做到更便宜快捷。

许院士将此引申到北斗设计上的重要不利因素：

- 在没有遍布世界的军事基地的情况下(要建立对应的地面监测站)，促使我们尝试更好的星座设计增加追踪弧度；
- 高精度的星载原子钟的技术被封锁，我们则试着通过频繁校对做相应补偿，为我们自主的原子钟研发争取时间；

而北斗性能的相对超越体现也有一些，比如：

- 北斗2号一期的卫星利用率可以达到 80 %，即使在建设初期，投入性价比也是很高的；
- 通过额外的同步卫星(二级监控站)来实现系统级的广域差分系统；
- 具有按需划配的位置主动上报功能；

我想起经济学中有一个重要的原理叫做“比较优势”(The principle of comparative advantage): 要将有限的资源去生产对自己来说机会成本更低的产品，然后通过交换获取其他所需，如此可以达到整个社会产品总价值最大。但遵从原理就能实现国富民强？改革开放初期缺资金、少技术，没人才，如何评估机会成本最低的项目，是不是只能搞“来料加工”？如何才能保证自己不被锁死在“劳动密集型低端制造”这个“比较优势”中无法自拔? 更重要的是谁来保证全世界的每个人/公司/国家遵从的最高原则就是将全社会的产品总价值达到最大？世界有时没有想象中理性，有时候就算你选了一个对自己来说最具比较优势的产业，你还是得问问大家是否答应让你做……

确实，要是只抱着“若无法全面超越碾压就别瞎折腾”的想法，放弃在这些自己原本不擅长的行业中学习、跟跑、积累，指望在新机会到来后临时抱佛脚，进而完成替代和赶超才是痴人说梦。

- 液晶面板，日本曾经占全行业9成以上份额，但现在却被韩国三星、LG掀翻在地，而我国的京东方已成为后起之秀;
- 通信设备，若没有近年来华为、中兴的强有力竞争，国外的传统通信巨头现在日子怕是仍然过的很滋润；
- 大飞机，据说巴西的支线飞机，进展很不错，我们呢？是花银子把飞机租借进行到底还是发展自己的系统集成能力？
- 航天卫星，当年我们和苏联关系好，但“科罗廖夫”们能帮我们造火箭、发卫星么？高铁呢？航母呢？

已然成为“世界工厂”的当下，我们再分析自己的比较优势，那些对我们机会成本最低的领域是不是已经转向到高端制造、软件服务等领域？而原有产业不管做得多么顺手，也要考虑主动转出了？

我把自己之前看到的一些行业分析的文章，列到了下面的扩展阅读中，真希望能找到视角同样宏大，论证翔实，但观点相异的更多靠谱文章。

## 拓展阅读

> - ConEmu [Bash on Windows](https://conemu.github.io/en/BashOnWindows.html)
> - The Debian GNU/Linux FAQ [Chapter 8 - The Debian package management tools](https://www.debian.org/doc/manuals/debian-faq/ch-pkgtools.en.html)
> - (2017-12-02) Abhishek Prakash [Difference Between apt and apt-get Explained](https://itsfoss.com/apt-vs-apt-get-difference/)
> - SO answer from drysdam [How do I get a list of installed files from a package?](https://askubuntu.com/a/32509)
> - 2016-09-27 packagecloud [Fixing APT Hash Sum Mismatch: Consistent APT Repositories](https://blog.packagecloud.io/eng/2016/09/27/fixing-apt-hash-sum-mismatch-consistent-apt-repositories/)
> - 知乎 [北斗有 35 颗卫星，而 GPS 有 24 颗卫星，为什么二者数量不同？](https://www.zhihu.com/question/21092045)
> - 宁南山 (2017-09-03) [又一个时代的结束--走向消亡的日本显示面板产业](https://zhuanlan.zhihu.com/p/29009696) 
> - 宁南山 (2018-05-20) [技术与国家--5G eMBB编码投票战始末](https://zhuanlan.zhihu.com/p/37060223)
> - 宁南山 (2017-05-18) [大飞机是中国人开辟的新战场](https://zhuanlan.zhihu.com/p/26755740)
> - 宁南山 (2018-02-08) [SpaceX猎鹰重型火箭升空与中美差距](https://zhuanlan.zhihu.com/p/33707817) 
> - 察网 刘枫 (2017-09-01) [前铁道部长傅志寰两篇重磅文章，揭示了我国高铁成功的真正原因](http://www.cwzg.cn/politics/201709/38216.html)
> - 知乎 Nibelung [巴西航空制造业比中国强吗？](https://www.zhihu.com/question/27406657/answer/252504498) 的回答
> - 秋山 (2018-03-23) [旧日本海军金刚型战列舰 舰政篇](https://zhuanlan.zhihu.com/p/34824855)

封面图片来自 [Satellite](https://dribbble.com/shots/3358840-Satellite) <a href="https://dribbble.com/septemcollium33"><i class="fa fa-dribbble" aria-hidden="true"></i>David Falter</a>