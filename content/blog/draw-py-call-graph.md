+++
date = "2017-11-24T11:37:35+08:00"
title = "“调用”可视化"
showonlyimage = false
image = "/img/blog/draw-py-call-graph/schema.png"
topImage = "/img/blog/draw-py-call-graph/schema.gif"
draft = true
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

{{< highlight python >}}
def f():
    for i in range(10):
        g(i)

def g(n):
    return h(n) + i(n)

def h(n):
    return n + 10

def i(n):
    return n * 8


if __name__ == '__main__':
    import trace
    
    t = trace.Trace(count=1, trace=0, countfuncs=1, countcallers=1)
    t.runfunc(f)
    r = t.results()
    print(dir(r))
    print(r.callers)

{{< /highlight >}}

上面跟踪了函数逐条语句执行情况，我们还可以通过向 Trace 传入其他各种开关跟踪函数调用情况。这样产生的 CoverageResults 对象的 key 也会包含不同粒度对象的调用情况。CoverageResults 的类型貌似总是 dict，其值也总为被调用次数。而 key 就因情况而异了，本例中为 tuple of tuples，第一层 tuple 为 (caller, callee) ，然后 caller 和 callee 又都为 tuple 起组成为 (filename, module, funcname)

> python 一直以文档完善而著称，但关于 trace 模块的文档却不是很详尽，你需要使用 python 的各种自省查看其内部的数据结构。这种不详尽的主要原因可能是因为只有资深开发者会使用这个模块追踪代码调用情况。或者其数据结构仍可能在未来变化?

### JSPlumb

这个 JS 库主要用于连接各种元素，其基础概念有

- Anchor 用户无法自行创建，但它标识了某一元素的 Endpoints 可存在的位置
- Endpoint 代表一个连接(Connection) 的某一端
- Connector 代表一个连接两个元素的连线
- Overlay 可用于装饰连线(Connector)，比如箭头、文本框
- Connection = Endpoint + Connector + Endpoint + Overlay

API 都暴露在 Connection 和 Endpoint 上，但你最终需要通过它们调整 Anchor, Connector 和 Overlay。

相比 nodejs 官方的 npm 工具，更推荐使用 Facebook 出品的包管理工具 yarn : 

- 速度快
- 去除不不必要的选项、指令短
- 支持缓存离线安装

> bower 项目已经停止。

{{< highlight console >}}
$ pacman -Q nodejs npm
nodejs 9.2.0-1
npm 5.5.1-2

$ pacman -Ss yarn
community/yarn 1.3.2-1
    Fast, reliable, and secure dependency management
$ pacman -S yarn

$ yarn config set proxy http://ip:port
$ yarn config set https-proxy http://ip:port

$ yarn info jsplumb
$ yarn add jsplumb

{{< /highlight >}}


参考文档

> - Michel J. Anders 
    - (2011-05-01) [Coverage Testing in Python: Combining the unittest and trace Modules](http://michelanders.blogspot.fi/2011/05/coverage-testing-in-python-combining.html) 
    - (2011-05-29) [Visualizing Python Call Graphs With Jsplumb](http://michelanders.blogspot.fi/2011/05/visualizing-python-call-graphs-with.html)
> - Dilinl Mampitiya (2016-03-04) [Implement a Flowchart Editor using jsPlumb – Part 1](https://dilinimampitiya.wordpress.com/2016/03/04/implement-a-flowchart-editor-using-jsplumb-part-1/)
> - Free Dev Tutorials [jsPlumb Tutorial](http://www.freedevelopertutorials.com/jsplumb-tutorial/)
> - Real Python (2014-05-03) [Primer on Jinja Templating](https://realpython.com/blog/python/primer-on-jinja-templating/)
> - [npm Tutorial for Beginners](https://www.youtube.com/playlist?list=PLC3y8-rFHvwhgWwm5J3KqzX47n7dwWNrq)
> - [Configuring a corporate proxy](http://www.jhipster.tech/configuring-a-corporate-proxy/)
> - [Creating Network Graphs with Python](https://www.udacity.com/wiki/creating-network-graphs-with-python)

封面图片来自 [Information design](https://dribbble.com/shots/2235198-Information-design-Theatre-Personal-project-WIP) <a href="https://dribbble.com/federicafragapane"><i class="fa fa-dribbble" aria-hidden="true"></i> Federica Fragapane</a>
