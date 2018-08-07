+++
date = "2018-08-01T23:05:11+08:00"
title = "ARTS 2018w31"
showonlyimage = false
image = "/img/blog/arts-05-per-week/chaos.png"
topImage = "/img/blog/arts-05-per-week/chaos.gif"
draft = false
weight = 1831
+++

找一个“杠精”帮助你设计、执行测试
<!--more-->

## Algorithm

本周题目 [1. Two Sum](https://leetcode.com/problems/two-sum/description/) 返回数组中的两个元素索引位置，使得他们的和为指定值(原数组中一定有解且唯一）。

{{< highlight python >}}
def twoSum(nums, target):
    buff_dict = {}
    for i in range(len(nums)):
        if nums[i] in buff_dict:
            return [buff_dict[nums[i]], i]
        else:
            buff_dict[target - nums[i]] = i
{{< /highlight >}}

关键在于利用 dict 设计一个缓存，一方面保存待寻找的另一个加数，另一方面同时保存当前加数的索引位置。

测试用例

{{< highlight python >}}
def test_two_sum(random_chk=1000):
    for _ in range(random_chk):
        len_a = randint(3, 100)
        input_a = [randint(0, 10) for _ in range(len_a)]

        m = randint(50, 60)
        n = randint(70, 80)

        m_idx = randint(0, len_a)
        n_idx = randint(0, len_a)

        input_a.insert(m_idx, m)
        input_a.insert(n_idx, n)

        if m_idx >= n_idx:
            m_idx += 1

        i, j = two_sum(input_a, m + n)
        assert input_a[i] + input_a[j] == m + n
        assert {i, j} == {m_idx, n_idx}
{{< /highlight >}}
<br/>

## Review 

本周读了来自 Thomas Guest "Word Aligned" 上的两篇文章:

- 第一篇 [Slicing a list evenly with Python](http://wordaligned.org/articles/slicing-a-list-evenly-with-python) (2017-5-14)  一步步地实现如何将一个列表尽可能的做 n 等分;
- 第二篇 [Unleash the test army](http://wordaligned.org/articles/unleash-the-test-army) (2017-05-29) 则利用 python 的 hypothesis 模块对上面的解法进行测试;

### 初始解法

{{< highlight python >}}
def chunk(xs, n):
    '''Split the list, xs, into n chunks
    '''
    L = len(xs)
    assert 0 < n <= L
    s = L//n
    return [xs[p:p+s] for p in range(0, L, s)]
{{< /highlight >}}

这个解法在当 xs 的长度 L 不是 n 的整数倍时，会得到 **n + 1** 个或 **L** 个片段(当 L // n == 1 且 L % n != 0 时 ）。

### 改进解法

尝试修复的算法如下——即将剩余元素放入最后一个 chunk

{{< highlight python >}}
def chunk(xs, n):
    '''Split the list, xs, into n chunks
    '''
    L = len(xs)
    assert 0 < n <= L
    s, r = divmod(L, n)
    chunks = [xs[p:p+s] for p in range(0, L, s)]
    chunks[n-1:] = [xs[-r-s:]]
    return chunks
{{< /highlight >}}

但问题是这样一来元素分布就不平均了——最后一段总是多出 r 个元素。

一种解决办法是生成 **r** 个含有 s + 1 个元素的片段，以及 **n - r** 个含 s 个元素的片段

{{< highlight python >}}
def chunk(xs, n):
    '''Split the list, xs, into n chunks
    '''
    L = len(xs)
    assert 0 < n <= L
    s, r = divmod(L, n)
    t = s + 1
    return ([xs[p:p+t] for p in range(0, r*t, t)] +
            [xs[p:p+s] for p in range(r*t, L, s)])

{{< /highlight >}}

### 传统测试

然后是一些一般意义上的测试用例。
{{< highlight python >}}
def test_chunk():
    with pytest.raises(AssertionError):
        assert chunk('', 1) == ['']
    assert chunk('ab', 2) == ['a', 'b']
    assert chunk('abc', 2) == ['ab', 'c']
    
    xs = list(range(8))
    assert chunk(xs, 2) == [[0, 1, 2, 3], [4, 5, 6, 7]]
    assert chunk(xs, 3) == [[0, 1, 2], [3, 4, 5], [6, 7]]
    assert chunk(xs, 5) == [[0, 1], [2, 3], [4, 5], [6], [7]]
    
    rs = range(1000000)
    assert chunk(rs, 2) == [range(500000), range(500000, 1000000)]
{{< /highlight >}}

### “新派”测试

但上面这些测试用例的特点是因人而异。但如果是我们只负责描述被测系统的属性，然后测试工具按照这些约束“召唤”出千奇百怪的输入，再看被测系统能否禁受的住的这样的自动测试大军的冲击。

这主意听起来不错，但这个“智能”的测试工具/框架能否达到预期效果依然存疑，但同样的，我们也该对传统测试集的完备性同样存疑……幸好 python 的 hypothesis 开了一个不错的头。

回到本题目的基础约束：

- 最后分好的段的数量为 n
- 各段长度不能参差不齐，要么全一样，要么一大一小（仅相差1）

委托 hypothesis 提供满足上述约束的输入测试集:

{{< highlight python >}}
import hypothesis as ht
import hypothesis.strategies as st

@ht.given(xs=st.lists(), n=st.integers())
def test_evenly_chunked(xs, n):
    chunks = chunk(xs, n)
    assert len(chunks) == n
    chunk_lens = {len(c) for c in chunks}
    assert len(chunk_lens) in {1, 2}
    assert max(chunk_lens) - min(chunk_lens) in {0, 1}
{{< /highlight >}}

然而函数在测试某些“犄角旮旯”的用例时报错：

- 将 xs 设为空列表
- 将 n 设为 0

这时作者做了如下几个设计上的选择：

1. 不再通过产生运行时的 AssertionError 对输入参数做合法性判定
2. 要支持 n > L 的情况
3. 不需要支持 n == 0 ，即便 xs 为空列表

{{< highlight python >}}
@ht.given(xs=st.lists(st.integers()),
          n=st.integers(min_value=1))
def test_evenly_chunked(xs, n):
    chunks = chunk(xs, n)
    # Same assert code as previous
{{< /highlight >}}

### 终极解法

{{< highlight python >}}
from itertools import accumulate, chain, repeat, tee

def chunk(xs, n):
    '''Split the list, xs, into n evenly sized chunks'''
    assert n > 0
    L = len(xs)
    s, r = divmod(L, n)
    widths = chain(repeat(s+1, r), repeat(s, n-r))
    offsets = accumulate(chain((0,), widths))
    b, e = tee(offsets)
    next(e)
    return [xs[s] for s in map(slice, b, e)]
{{< /highlight >}}

再次运行测试时 hypothesis 在模拟 n 特别大的情况时，因为需要太多资源，函数无法中止。

### 终极测试

解决方式一种是将原函数改为 python generator ，另一种方式认为这种情况不应该发生，约束 hypothesis 测试时不产生过大的 n 。

{{< highlight python >}}
@ht.given(xs=st.one_of(st.text(),
                       st.binary(),
                       st.lists(st.integers())),
          n=st.integers(min_value=1, max_value=100))
def test_evenly_chunked(xs, n):
    chunks = chunk(xs, n)
    # Same assert code as previous
{{< /highlight >}}


{{< highlight python >}}
import functools

import hypothesis as ht
import hypothesis.strategies as st

@st.composite
def items_and_chunk_count(draw):
    xs = draw(st.one_of(st.text(),
                        st.binary(),
                        st.lists(st.integers())))
    n = draw(st.integers(min_value=1,
                         max_value=max(1, len(xs))))
    return xs, n

@ht.given(xs_n=items_and_chunk_count())
def test_evenly_chunked(xs_n):
    '''Verify there are n evenly sized chunks'''
    xs, n = xs_n
    chunks = chunk(xs, n)
    assert len(chunks) == n
    chunk_lens = {len(c) for c in chunks}
    assert len(chunk_lens) in {1, 2}
    assert max(chunk_lens) - min(chunk_lens) in {0, 1}

@ht.given(xs_n=items_and_chunk_count())
def test_combining_chunks(xs_n):
    '''Verify recombining the chunks reproduces the original sequence.'''
    xs, n = xs_n
    chunks = chunk(xs, n)
    assert functools.reduce(lambda x, y: x+y, chunks) == xs

{{< /highlight >}}
{{< highlight console >}}
λ pipenv run pytest --hypothesis-show-statistics
=================== test session starts ===================
platform win32 -- Python 3.6.5, pytest-3.7.1, py-1.5.4, pluggy-0.7.1
rootdir: ~\code\leetcode\slicing_a_list_evenly, inifile:
plugins: cov-2.5.1, hypothesis-3.66.23
collected 3 items

test_solution.py ...                                                     [100%]
================== Hypothesis Statistics ==================

test_solution.py::test_evenly_chunked:

  - 100 passing examples, 0 failing examples, 0 invalid examples
  - Typical runtimes: 0-15 ms
  - Fraction of time spent in data generation: ~ 40%
  - Stopped because settings.max_examples=100

test_solution.py::test_combining_chunks:

  - 100 passing examples, 0 failing examples, 0 invalid examples
  - Typical runtimes: 0-16 ms
  - Fraction of time spent in data generation: ~ 54%
  - Stopped because settings.max_examples=100

================ 3 passed in 0.93 seconds =================
{{< /highlight >}}

总结下来:

- 强制我们去思考确认被测对象的规范
- 产生各种作者可能想不到的逻辑上合规的输入，测试被测对象的行为
- 测试和具体实现最低程度的耦合，方便代码重构——用完全不同的思路重新实现需求

基于特性的测试(property based testing)和基于示例的测试(example based testing)互为补充。虽然前者在设置数据生成策略和定义系统约束上会很困难，但这仍值得尝试。

就像 Hypothesis 作者说的那样：“人们都说，软件正在吞噬世界。但同时软件也糟透了，富含 bug、冲动而危险、禁不起太多的思考推敲——这一切相互作用泡制出一口灾难火锅。而软件测试的现状更糟，扪心自问，我们开发的系统有多少是经过充足测试的？写出好的测试非常难，一个重要原因是你基于你当初写代码时脑中的假设/谬误写出这些测试，所以他们测不出问题。”
<br/>

## Tip

有时候希望对电脑中播放的媒体进行录音，利用 Windows 的 WASAPI (Windows Audio Session API) 特性配合 Audacity 一起完成。试了一下，效果还不错。具体操作见 Chris Hoffman (2017-10-24) [How to Record the Sound Coming From Your PC (Even Without Stereo Mix)](https://www.howtogeek.com/217348/how-to-record-the-sound-coming-from-your-pc-even-without-stereo-mix/)

{{< highlight shell >}}
choco install audacity
choco install audacity-lame
{{< /highlight >}}

对于录屏，CamStudio 处于濒死的状态？而 Active Presenter 看上去值得尝试。

{{< highlight shell >}}
choco install activepresenter
{{< /highlight >}}
<br/>

## Share

上文提到的分割问题: 如果不考虑对元素做连续分割——本来题目中也没有要求这一点——那么会有非常简单优雅的解决算法：

{{< highlight python >}}
def chunk(xs,n):
    return [xs[index::n] for index in range(n)]
{{< /highlight >}}

但是上面的 ```test_combining_chunks``` 需要做相应改动。

或者说如果你熟悉 numpy ，直接使用 np.array_split 则根本不需要重新造轮：

{{< highlight python >}}
>>> from numpy import array_split as chunk
>>> xs = list('ab')
>>> chunk(xs, 5)
[array(['a'], dtype='|S1'), array(['b'], dtype='|S1'), array([], dtype='|S1'), array([], dtype='|S1'), array([], dtype='|S1')]
{{< /highlight >}}

之前的一通操作和学习可谓是“在平坦的路上曲折前行了”，但下面的“坑”也应该“填平”:

1. 找到 np.array_split 的源码，看人家是如何实现的
2. 通过压力测试比较上述几种实现的执行效率
3. 以往做过的 leetcode 的题目是否可以用 hypothesis 进行测试

<br/>

## 拓展阅读

Thomas Guest @ Word Aligned

> - (2007-11-20) [Zippy triples served with Python](http://wordaligned.org/articles/zippy-triples-served-with-python)
> - (2016-3-21) [Sausages, sausages, sausages - slice, slice, slice](http://wordaligned.org/articles/sausages-slices)


ScottW 

> - (2014-12-01) [An introduction to property-based testing](https://fsharpforfunandprofit.com/posts/property-based-testing/)
> - (2014-12-12) [Choosing properties for property-based testing](https://fsharpforfunandprofit.com/posts/property-based-testing-2/)

封面图片来自 [Chaos Reigns](https://dribbble.com/shots/1840925-Chaos-Reigns) <a href="https://dribbble.com/bushra"><i class="fa fa-dribbble" aria-hidden="true"></i> Bushra Mahmood</a>