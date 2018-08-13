+++
date = "2017-09-08T23:27:52+08:00"
title = "python 并发 (1)"
showonlyimage = false
image = "/img/blog/intro-to-py-socket/sorting_kit8-net.png"
draft = false
weight = 101
tags = ["python", "async"]
+++

[David Beazley](http://www.dabeaz.com/) 的[直播讲解](https://www.youtube.com/watch?v=MCs5OvhV9S4) Socket 并发
<!--more-->

本文介绍了 socket 编程的阻塞问题，多线程的 GIL 问题，传统的线程池解决方案。

免责声明: 因为代码是 dabeza 在 2015 Pycon 上的现场演示，所以他特别提示观众他为了将 Python 并发的最核心概念在短短 40 分钟内展现出来，不得不放弃了软件工程上所有的最佳实践、防御性编程。

## 兔子数列

斐波那契数列的一个简陋实现可能是这个样子的：
{{< highlight python >}}
def fib(n):
    if n <= 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)
{{< /highlight >}}

{{< highlight console >}}
$ python3
Python 3.6.2 (default, Jul 20 2017, 03:52:27)
[GCC 7.1.1 20170630] on linux
Type "help", "copyright", "credits" or "license" for more info  
>>> from fib import fib
>>> fib(1)
1
>>> fib(10)
55
>>> fib(20)
6765
{{< /highlight >}}

随着求解的数列位置越来越靠后，运算的时间也会指数级加长。

## 微服务版本

{{< highlight python "linenos=table">}}
# server.py
# Fib microservice

from socket import *
from fib import fib


def fib_server(address):
    sock = socket(AF_INET, SOCK_STREAM)
    sock.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)
    sock.bind(address)
    sock.listen(5)
    while True:
        client, addr = sock.accept()
        print("Connected ", addr)
        fib_handler(client)


def fib_handler(client):
    while True:
        req = client.recv(100)
        if not req:
            break
        n = int(req)
        result = fib(n)
        resq = str(result).encode('ascii') + b'\n'
        client.send(resp)
    print("Closed")


fib_server(('', 25000))

{{< /highlight >}}

socket 算是 python 网络编程领域较为底层的模块。

第 9 行初始化的时候```socket(AF_INET, SOCK_STREAM)```可指定套接字(插座)的传输协议，这里两个参数都是默认值——不明写也行```sock = socket()```。AF 就是 Addr Family，第一个参数 AF_INET ≃ IPv4 。第二个参数 SOCK_STREAM 用于 TCP，与之相对还有 SOCK_DGRAM 用于 UDP。

第 10 行设置 Socket Level 的选项，将 SO_REUSEADDR 置为 yes 。主要是因为后续我们将不断调试程序，如此设置使得即便杀掉服务端程序后立即重启也不会出现因为连接拒绝建立的情况。详见 [What is the meaning of SO_REUSEADDR (setsockopt option) - Linux](https://stackoverflow.com/questions/3229860/what-is-the-meaning-of-so-reuseaddr-setsockopt-option-linux)

第 11 行给出一个 socket 五元组的其中二项，即服务器端的网络地址和端口:{PROTOCOL, SRC-IP, SRC-PORT, DEST-IP, DEST-PORT}

第 12 行让服务端开始监听来自客户端(插头)的连接。参数选填，及连接队列的最大槽位。

第 14 行是个阻塞调用，仅在有客户端连接时返回 (conn, address)，其中 conn 是一个新的 socket 对象，专用于和此客户端收、发消息。addr 就是客户端的网络地址了。

第 21 行的 recv 同样是个阻塞调用，专用于 TCP 连接，参数为一次收下的最大数据长度。建议取值是不太大的 2 的指数幂。

第 27 行的 send 是阻塞调用，也专用于 TCP 连接。

{{< highlight console >}}

tty1 $ python3 server.py
Connected  ('127.0.0.1', 46310)

tty2 $ telnet localhost 25000 # netcat/nc also works
1
1
10
55
20
6765
{{< /highlight >}}

但问题是这个服务器版本不能同时为多个 client 服务。这是由于代码仅由[单一处理进程处理所有请求](https://stackoverflow.com/a/11344212/4393386)，只有 client 2 参与建立的 socket 链接（五元组）拆链后，才能继续处理早已处于等待状态的来自 client 3 的另一个 socket 链接。

{{< highlight console >}}
tty3 $ telnet localhost 25000
10
(hung with no response unless tty2 kill its connection)
{{< /highlight >}}

<img alt="Py Socket Workflow" src="/img/blog/intro-to-py-socket/Py-Socket-WorkFlow.png"  style="width:70%; height:70%; display:block; margin: auto;">

需要引入多线程，才能实现为多个 client 同时服务。

> ```git format-patch -n HEAD^``` 或 ```git show HEAD```生成或查看 delta

{{< highlight diff >}}
...
Subject: [PATCH 1/1] use thread to support multi clients

---
 server.py | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)

diff --git a/server.py b/server.py
index 8e98d26..25aed6f 100644
--- a/server.py
+++ b/server.py
@@ -3,6 +3,7 @@

 from socket import *
 from fib import fib
+from threading import Thread


 def fib_server(address):
@@ -13,7 +14,7 @@ def fib_server(address):
     while True:
         client, addr = sock.accept()
         print("Connected ", addr)
-        fib_handler(client)
+        Thread(target=fib_handler, args=(client,), daemon=True).start()


 def fib_handler(client):
--
2.14.1
{{< /highlight >}}

{{< highlight console >}}
tty1 $ python3 server.py
Connected  ('127.0.0.1', 50142)
Connected  ('127.0.0.1', 50146)

tty2 $ telnet 127.0.0.1 25000
10
55
20
6765

tty3 $ telnet 127.0.0.1 25000
20
6765
30
832040

{{< /highlight >}}

## GIL 性能瓶颈

CPython 的 thread 直接使用操作系统上的线程，但因为 python 的内存管理并非线程安全，需要借助 GIL (globla interpreter lock)，会导致一些性能上的问题。

比如，下面的程序持续地向服务器发出计算请求:

{{< highlight python >}}
# perf1.py
# Time of a long running request

from socket import *
import time

sock = socket(AF_INET, SOCK_STREAM)
sock.connect(('localhost', 25000))

while True:
    start = time.time()
    sock.send(b'30')
    resp = sock.recv(100)
    end = time.time()
    print(end-start)
{{< /highlight >}}

如果单独执行，平均处理时间大概是 1/5 秒。

{{< highlight console >}}
tty 4 $ python3 perf1.py
0.26836633682250977
0.24056386947631836
0.23276972770690918
0.23243498802185059
0.23028111457824707
0.22887349128723145
0.2297065258026123
0.22983860969543457
0.22971153259277344

{{< /highlight >}}

但如果同时启动 n 个这样的 clients，处理时长就会(线性?)增长——所有请求仅能经单核处理。David Beazley 演示的时候，其 Mac 确实是线性增长。但我实验的结果性能下降更多(5倍)，不知是不是我后台运行程序太多的缘故。

{{< highlight console >}}
tty 5 $ python3 perf1.py
1.29852294921875
1.2276148796081543
1.3492326736450195
1.3649957180023193

tty 4
...
0.22785305976867676
0.2288191318511963
0.2296619415283203
1.1391808986663818
1.277223825454712
1.2258660793304443
1.0844104290008545
1.3383533954620361

{{< /highlight >}}

## GIL 优先级瓶颈

另一个现象可以借助下面的程序展示。在tty2上用多线程模拟密集而运算量极小的 client，得到当前机器平均每秒可以处理的请求数量。我实验的结果是 2～3 万请求每秒。

然后再在 tty 3 上启动一个 telnet ，单独提交一个请求，比如 42。你会发现原来 tty 2 上每秒处理的请求瞬间下降了100倍（从2～3万到100）——几乎是个断崖式的下降，而直到这个单独的请求被处理完毕，才恢复回之前每秒处理量级。

{{< highlight python >}}
# perf2.py
# requests/sec of fast requests

from socket import *
import time


sock = socket(AF_INET, SOCK_STREAM)
sock.connect(('localhost', 25000))

n = 0


from threading import Thread
def monitor():
    global n
    while True:
        time.sleep(1)
        print(n, 'reqs/sec')
        n = 0
Thread(target=monitor).start()


while True:
    sock.send(b'1')
    resp = sock.recv(100)
    n += 1
{{< /highlight >}}

{{< highlight console >}}
tty 2 $ python3 perf2.py
35318 reqs/sec
33660 reqs/sec
25461 reqs/sec
22900 reqs/sec
10340 reqs/sec
224 reqs/sec
108 reqs/sec
100 reqs/sec
6402 reqs/sec
28553 reqs/sec
20865 reqs/sec
21104 reqs/sec
31101 reqs/sec

tty 3 $ telnet localhost 25000
36
14930352

{{< /highlight >}}

这个现象背后原因是 GIL 有个特性是其内部会做优先级排序，将计算量较大的设为高优先级，但这种行为其实和操作系统产生了较大差别。

假设通过如下这种方式在 tty 3 上启动另一个线程出发计算，原有的 tty 2 上的压测数值就不会发生变化。就是说操作系统其实更愿意为短小计算设置高优先级。

{{< highlight console >}}
tty 3 $ python3 -i fib.py
>>> fib(36)
14930352

tty2
32687 reqs/sec
38291 reqs/sec
40569 reqs/sec
34638 reqs/sec
36722 reqs/sec
35463 reqs/sec
30825 reqs/sec
28908 reqs/sec
33111 reqs/sec

{{< /highlight >}}

想象你开设了类似的 Web 服务，大部分的请求时短小的，但忽然的一个大计算量的请求会把整个系统服务的请求数压到非常低。

一般性的解决方案是使用进程池，这样每秒能处理的小请求的数量会减少 1/10 (从2～3万到2千)——因为 Pool 引入了不少间接成本，但好处是将由于 GIL 的任务优先级特性，在大运算量请求来临时影响降到了很小。

{{< highlight diff >}}

$ git diff
diff --git a/server.py b/server.py
index 25aed6f..61885ed 100644
--- a/server.py
+++ b/server.py
@@ -4,7 +4,9 @@
 from socket import *
 from fib import fib
 from threading import Thread
+from concurrent.futures import ProcessPoolExecutor as Pool

+pool = Pool(4)

 def fib_server(address):
     sock = socket(AF_INET, SOCK_STREAM)
@@ -23,7 +25,8 @@ def fib_handler(client):
         if not req:
             break
         n = int(req)
-        result = fib(n)
+        future = pool.submit(fib, n)
+        result = future.result()
         resq = str(result).encode('ascii') + b'\n'
         client.send(resq)
     print("Closed")

{{< /highlight >}}

{{< highlight console >}}
tty2
...
1829 reqs/sec
2090 reqs/sec
2124 reqs/sec
2161 reqs/sec
2110 reqs/sec
2174 reqs/sec
...

tty 3 $ telnet localhost 25000
36
14930352

{{< /highlight >}}

<br />

总结

1. 首先通过引入 socket 将 fib 的计算做成服务  
2. 受限于 accpet、recv、send 均为 socket 的阻塞调用，只有当前请求的 socket 拆链后，才能服务于下一个请求
3. 引入线程模块，使得每一个请求都可以得到同时享受到服务
4. 受限于 python 的 GIL，类似 fib(30) 这样量级的计算请求，若同时持续触发，平均响应时间会线性增长
5. 同样受限于 GIL 的优先级特性，就算单独一个计算量较大的请求，如 fib(42) 会优先执行，而阻塞像 fib(1) 数量众多的小请求
6. 引入线程池因为引入了不小的系统开销成本，可以一定程度缓解系统被大计算请求阻塞的问题

篇幅有限，未完待续

参考文档

> - python wiki [GIL](https://wiki.python.org/moin/GlobalInterpreterLock)
> - David Beazley [code@github](https://github.com/dabeaz/concurrencylive)
> - Michael Domanski (2016-06-09) [How Celery fixed Python's GIL problem](http://blog.domanski.me/how-celery-fixed-pythons-gil-problem/)
> - Python Doc socket [Low-level networking itf](https://docs.python.org/3/library/socket.html)
> - [Socket options SO_REUSEADDR and SO_REUSEPORT, how do they differ? Do they mean the same across all major operating systems?](https://stackoverflow.com/questions/14388706/socket-options-so-reuseaddr-and-so-reuseport-how-do-they-differ-do-they-mean-t)
> - Harsh S. (2016-10-31) [Essentials Of Python Socket Programming You Should Know](http://www.techbeamers.com/python-tutorial-essentials-of-python-socket-programming/)
> - Silver Moon (2014-01-09) [Python socket – network programming tutorial](http://www.binarytides.com/python-socket-programming-tutorial/) and (2016-08-06) [Programming udp sockets in python](http://www.binarytides.com/programming-udp-sockets-in-python/)
> - M. Jones (2005-10-04) [Sockets programming in Python](https://www.ibm.com/developerworks/linux/tutorials/l-pysocks/index.html)

封面图片来自 [Sorting factory](https://dribbble.com/shots/3326497-Sorting-factory) <a href="https://dribbble.com/Frizler"><i class="fa fa-dribbble" aria-hidden="true"></i> Anton Fritsler (kit8)</a>
