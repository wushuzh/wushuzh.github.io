+++
date = "2017-09-21T20:19:58+08:00"
title = "python 并发 (3)"
showonlyimage = false
Image = "/img/blog/intro-to-py-3/seatedsuperman.png"
topImage = "/img/blog/intro-to-py-3/seatedsuperman.gif"
draft = false
weight = 101
+++

最终版本的 AsyncSocket 及总结
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

可以看出，当 fib(36) 执行期间，perf2 客户端上的计算被完全阻隔，完全无法为其提供任何服务。这在本系列的第一篇的最后解释了这是因为 GIL 的任务优先级设定的特性决定的。

## 新的阻塞

这里的解决办法和之前一样: 使用线程池。

{{< highlight diff >}}

--- a/aserver.py
+++ b/aserver.py
@@ -5,11 +5,14 @@ from socket import *
 from fib import fib
 from collections import deque
 from select import select
+from concurrent.futures import ThreadPoolExecutor as Pool

 tasks = deque()
 recv_wait = { }   # Mapping sockets -> tasks (generators)
 send_wait = { }

+pool = Pool(10)
+
 def run():
     while any([tasks, recv_wait, send_wait]):
         while not tasks:
@@ -54,7 +57,8 @@ def fib_handler(client):
         if not req:
             break
         n = int(req)
-        result = fib(n)
+        future = pool.submit(fib, n)
+        result = future.result()  # blocking
         resq = str(result).encode('ascii') + b'\n'
         yield 'send', client
         client.send(resq)    # blocking

{{< /highlight >}}

但如果你仅在此处重复测试一下 perf2，观察到的结果是其每秒服务请求仍然会降到0，问题并没有解决。原因是 ```future.result()``` 是被新的线程池代码引入的一个阻塞调用。

即如果你使用协程，你就要全部使用。如果你不知道哪些调用是阻塞性的，并作出相应修改，那么最终性能必然会被这些调用阻塞，而无法达到预期结果。

## 魔法时刻

解决方案是为 ```future``` 修补**事件循环调度器**。

{{< highlight diff "linenos=table">}}
--- a/aserver.py
+++ b/aserver.py
@@ -10,6 +10,20 @@ from concurrent.futures import ThreadPoolExecutor as Pool
 tasks = deque()
 recv_wait = { }   # Mapping sockets -> tasks (generators)
 send_wait = { }
+future_wait = { }
+
+future_notify, future_event = socketpair()
+
+def future_done(future):
+    tasks.append(future_wait.pop(future))
+    future_notify.send(b'x')
+
+def future_monitor():
+    while True:
+        yield 'recv', future_event
+        future_event.recv(100)
+
+tasks.append(future_monitor())

 pool = Pool(10)

@@ -32,6 +46,10 @@ def run():
                 recv_wait[what] = task
             elif why == 'send':
                 send_wait[what] = task
+            elif why == 'future':
+                future_wait[what] = task
+                what.add_done_callback(future_done)
+
             else:
                 raise RuntimeError("ARG!")
         except StopIteration:
@@ -58,6 +76,7 @@ def fib_handler(client):
             break
         n = int(req)
         future = pool.submit(fib, n)
+        yield 'future', future
         result = future.result()  # blocking
         resq = str(result).encode('ascii') + b'\n'
         yield 'send', client

{{< /highlight >}}

第 39 行: 依然是在阻塞调用前加入 yield ，目的是提示外层程序(第 28、29 行)——即将发生的阻塞是用于等待 future 的结果。继而向 future 注册```回调函数 future_done```。目的是当 future 计算完成，就将其从 ```等待区 future_wait``` 清出，重新放回 tasks 。

第 9 行创建了一个 Unix 域 Socket 名为 socketpair 两端分别为 (```future_notify```、```future_event```):

- 第 13 行通过 ```future_notify``` 向 socket 发送一个字符 x ;
- 第 15 ～ 20 行```监控函数 future_monitor```不断轮循查看 socket 另一端```future_event```是否接受到字符 x ——此任务也在程序初始时被加入到 tasks 中。

> 一个有趣的现象是如果使用 ProcessPoolExecutor 而非 ThreadPoolExecutor，我得到了一个诡异的现象

{{< highlight console >}}
tty2 $python3 perf2.py
746 reqs/sec
769 reqs/sec
728 reqs/sec
1492 reqs/sec
1689 reqs/sec
1675 reqs/sec
1683 reqs/sec
1674 reqs/sec
1149 reqs/sec
737 reqs/sec
795 reqs/sec
744 reqs/sec

tty3 $ telnet localhost 25000
36
14930352

{{< /highlight >}}

## 完整示例

最后我们通过定义类 AsyncSocket 封装原生 Socket 。

{{< highlight python >}}
# server.py
# Fib microservice

from socket import *
from fib import fib
from collections import deque
from select import select
from concurrent.futures import ThreadPoolExecutor as Pool
from concurrent.futures import ProcessPoolExecutor as Pool

pool = Pool(4)

tasks = deque()
recv_wait = { }   # Mapping sockets -> tasks (generators)
send_wait = { }
future_wait = { }

future_notify, future_event = socketpair()

def future_done(future):
    tasks.append(future_wait.pop(future))
    future_notify.send(b'x')

def future_monitor():
    while True:
        yield 'recv', future_event
        future_event.recv(100)

tasks.append(future_monitor())

def run():
    while any([tasks, recv_wait, send_wait]):
        while not tasks:
            # No active tasks to run
            # wait for I/O
            can_recv,can_send,_ = select(recv_wait,send_wait,[])
            for s in can_recv:
                tasks.append(recv_wait.pop(s))
            for s in can_send:
                tasks.append(send_wait.pop(s))


        task = tasks.popleft()
        try:
            why, what = next(task)   # Run to the yield
            if why == 'recv':
                # Must go wait somewhere
                recv_wait[what] = task
            elif why == 'send':
                send_wait[what] = task
            elif why == 'future':
                future_wait[what] = task
                what.add_done_callback(future_done)

            else:
                raise RuntimeError("ARG!")
        except StopIteration:
            print("task done")

class AsyncSocket(object):
    def __init__(self, sock):
        self.sock = sock
    def recv(self, maxsize):
        yield 'recv', self.sock
        return self.sock.recv(maxsize)
    def send(self, data):
        yield 'send', self.sock
        return self.sock.send(data)
    def accept(self):
        yield 'recv', self.sock
        client, addr = self.sock.accept()
        return AsyncSocket(client), addr
    def __getattr__(self, name):
        return getattr(self.sock, name)

def fib_server(address):
    sock = AsyncSocket(socket(AF_INET, SOCK_STREAM))
    sock.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)
    sock.bind(address)
    sock.listen(5)
    while True:
        client, addr = yield from sock.accept()  # blocking
        print("Connection", addr)
        tasks.append(fib_handler(client))

def fib_handler(client):
    while True:
        req = yield from client.recv(100)   # blocking
        if not req:
            break
        n = int(req)
        future = pool.submit(fib, n)
        yield 'future', future
        result = future.result()    #  Blocks
        resp = str(result).encode('ascii') + b'\n'
        yield from client.send(resp)    # blocking
    print("Closed")

tasks.append(fib_server(('',25000)))
run()

{{< /highlight >}}

TODO1: pydocs [asyncio Asynchronous I/O, event loop, coroutines and tasks](https://docs.python.org/3/library/asyncio.html)
TODO2: [A Curious Course on Coroutines and Concurrency](http://www.dabeaz.com/coroutines/)

参考文档

> - Python 3 docs [Launching parallel tasks](https://docs.python.org/3/library/concurrent.futures.html)
> - Masnun (2016-03-29）[Python: a quick intro to the concurrent.futures  module](http://masnun.com/2016/03/29/python-a-quick-introduction-to-the-concurrent-futures-module.html)
> - Unix 域套接字 [Unix domain socket](https://en.wikipedia.org/wiki/Unix_domain_socket)
> - Beej's Guide to Unix IPC (2015-12-01) [Chapter 11 Unix Sockets](http://beej.us/guide/bgipc/output/html/multipage/unixsock.html)
> - Troy D. Hanson (2015-02-16) [UNIX domain sockets](http://troydhanson.github.io/network/Unix_domain_sockets.html)

封面图片来自 [Seated Superman](https://dribbble.com/shots/2843135-Seated-Superman) <a href="https://dribbble.com/3rdfloor"><i class="fa fa-dribbble" aria-hidden="true"></i> Warren Willmott</a>
