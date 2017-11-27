+++
date = "2017-11-24T11:37:35+08:00"
title = "“调用”可视化"
showonlyimage = false
image = "/img/blog/draw-py-call-graph/schema.png"
topImage = "/img/blog/draw-py-call-graph/schema.gif"
draft = false
weight = 110
+++

追踪代码调用，生成 call grapy
<!--more-->

### 代码覆盖

前几天听了 Ned Batchelder (Coverage.py 的维护者）的[访谈](https://www.podcastinit.com/coverage-py-with-ned-batchelder-episode-121/)，也提到了 trace 模块可以用来跟踪代码执行覆盖情况。于是从网上找到两篇介绍追踪呈现方法调用关系的文章，见文末参考文档

首先写一个被测函数 f，并使用 unittest 对 f 的不同的输入类型和处理分支写测试用例。

{{< highlight python >}}
def f(a):
    if a > 0:
        return a
    elif a == 0:
        raise ValueError()
    return -a
# end of coverage

if __name__ == "__main__":
    import unittest

    class TestFuncF(unittest.TestCase):
        def test_int(self):
            self.assertEqual(3, f(3))
            self.assertEqual(3, f(-3))

        def test_float(self):
            self.assertEqual(3.0, f(3.0))
            self.assertEqual(3.0, f(-3.0))

{{< /highlight >}}

下一步，在 trace 框架下运行[测试用例](https://docs.python.org/3/library/unittest.html#unittest.main)，运行结束后获取跟踪结果: 从复杂的 [trace.CoverageResults](https://docs.python.org/3/library/trace.html#trace.CoverageResults) 中提取出各行代码运行次数，转存到 [defaultdict](https://docs.python.org/3/library/collections.html#defaultdict-objects) 中，因此其值一定是 int 。

{{< highlight python >}}
    import trace

    t = trace.Trace(count=1, trace=0)
    t.runfunc(unittest.main, exit=False)
    r = t.results()

    from collections import defaultdict

    linecount = defaultdict(int)
    for line in r.counts:
        #    line[0]     line[1]
        # { ('srcfile1', lineno): count, ... }
        if line[0] == __file__:
            linecount[line[1]] = r.counts[line]

{{< /highlight >}}

最后读取当前文件源码，找到函数 f 的各行代码执行次数并打印。这样就会发现有一个重要的分支没有覆盖到。

{{< highlight python >}}
    from re import compile, match

    eoc = compile(r'^\s*# end of coverage')
    with open(__file__) as f:
        for linenumber, line in enumerate(f, start=1):
            if eoc.match(line): break
            print(f"{linecount[linenumber]:02} {line}", end='')
{{< /highlight >}}

{{< highlight console >}}
..
-------------------------------------------
Ran 2 tests in 0.002s

OK
00 def f(a):
04     if a > 0:
02         return a
02     elif a == 0:
00         raise ValueError()
02     return -a

{{< /highlight >}}

### 调用关系

参考文档

> - Michel J. Anders 
    - (2011-05-01) [Coverage Testing in Python: Combining the unittest and trace Modules](http://michelanders.blogspot.fi/2011/05/coverage-testing-in-python-combining.html) 
    - (2011-05-29) [Visualizing Python Call Graphs With Jsplumb](http://michelanders.blogspot.fi/2011/05/visualizing-python-call-graphs-with.html)

封面图片来自 [Information design](https://dribbble.com/shots/2235198-Information-design-Theatre-Personal-project-WIP) <a href="https://dribbble.com/federicafragapane"><i class="fa fa-dribbble" aria-hidden="true"></i> Federica Fragapane</a>
