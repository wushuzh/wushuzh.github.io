+++
date = "2017-09-21T20:19:58+08:00"
title = "python 并发 (3)"
showonlyimage = false
Image = "/img/blog/intro-to-py-3/seatedsuperman.png"
topImage = "/img/blog/intro-to-py-3/seatedsuperman.gif"
draft = false
weight = 101
+++

观看牛人 [David Beazley](http://www.dabeaz.com/) 的[代码直播](https://www.youtube.com/watch?v=MCs5OvhV9S4)
<!--more-->

## 性能问题

上篇 ["python 并发 (2)"]({{< relref "blog/try-io-multiplexing-py.md" >}}) 最后实现的服务端版本已经能同时服务多个客户端，而不借助多线程。

本篇我们继续对其执行 ["python 并发 (1)"]({{< relref "blog/intro-to-py-socket.md" >}}) 中的性能测试。

首先是连续请求 fib(30) 的相应时长:

{{< highlight console >}}

tty1 $ python3 aserver.py
Connected  ('127.0.0.1', 33092)
Connected  ('127.0.0.1', 33104)
Closed
task done
Closed
task done

tty2 $ python3 perf1.py
0.2326054573059082
0.2293095588684082
0.24428391456604004
0.46734166145324707
0.4711148738861084
0.4698026180267334
0.4607887268066406
0.23572278022766113
0.23203778266906738
0.2307577133178711

tty3 $ python3 perf1.py
0.6155200004577637
0.4710536003112793
0.4698026180267334
0.460796594619751

{{< /highlight >}}

可以看出，我们依然被 GIL 困扰，相应时间随同时服务的 client 数量线性变大。

之后是每秒最大请求数在单个CPU密集请求时的变化:

{{< highlight console >}}

tty2 $ python3 perf2.py
30783 reqs/sec
33957 reqs/sec
8399 reqs/sec
0 reqs/sec
0 reqs/sec
0 reqs/sec
19083 reqs/sec
31683 reqs/sec
33548 reqs/sec

tty3 $ telnet localhost 25000
36
14930352

{{< /highlight >}}

可以看出，当 fib(36) 执行期间，perf2 客户端上的计算被完全阻隔，完全无法为其提供任何服务。

参考文档

> -

封面图片来自 [Seated Superman](https://dribbble.com/shots/2843135-Seated-Superman) <a href="https://dribbble.com/3rdfloor"><i class="fa fa-dribbble" aria-hidden="true"></i> Warren Willmott</a>
