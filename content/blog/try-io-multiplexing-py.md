+++
date = "2017-09-14T21:28:44+08:00"
title = "python 并发 (2)"
showonlyimage = false
image = "/img/blog/try-io-multiplexing-py/pantone_color_picker.png"
topImage = "/img/blog/try-io-multiplexing-py/pantone_color_picker.gif"
draft = false
weight = 101
+++

观看牛人 [David Beazley](http://www.dabeaz.com/) 的[代码直播](https://www.youtube.com/watch?v=MCs5OvhV9S4)
<!--more-->

## 生成器简介

在上篇 ["python 并发 (1)"]({{< relref "blog/intro-to-py-socket.md" >}}) ，我们之所以要引入线程，是因为下列三个阻塞调用：

- 用于等待 socket 连接的 accept
- 建立的 socket 连接后等待 client 发送时的 recv
- 发送给 client 回应时的 send

本篇我们尝试通过引入 geneartor —— 一个为方便定制循环而引入的概念，来解决上述阻塞问题。

首先复习 generator

{{< highlight console >}}
$ python3
Python 3.6.2 (default, Jul 20 2017, 03:52:27)
[GCC 7.1.1 20170630] on linux
Type "help", "copyright", "credits" or "license" for more
>>> def countdown(n):
...     while n > 0:
...         yield n
...         n -= 1
...
>>> c = countdown(5)
>>> print(c)
<generator object countdown at 0x7f23959ee468>
>>> for i in c:
...     print(i)
...
5
4
3
2
1
>>> c = countdown(3)
>>> next(c)
3
>>> next(c)
2
>>> next(c)
1
>>> next(c)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration

{{< /highlight >}}

下面示例中 run 方法的逻辑和典型的循环调度器的机制类似，以 round-robin 的方式轮循三个 generators 。

需要知晓的是 collections.deque 对象( double-ended queue ) 线程安全，无论基于头还是尾的 append 和 pop 都是 O(1)

{{< highlight console >}}

>>> from collections import deque
>>> tasks = deque()
>>>
>>> tasks.extend([countdown(5), countdown(8), countdown(3)])
>>> tasks
deque([<generator object countdown at 0x7f23959ee990>,
       <generator object countdown at 0x7f23959eea40>,
       <generator object countdown at 0x7f23959eef10>])

>>> def run():
...     while tasks:
...         task = tasks.popleft()
...         try:
...             x = next(task)
...             print(x)
...             tasks.append(task)
...         except StopIteration:
...             print("Task")
...
>>> run()
5
8
3    # l1
4
7
2    # l2
3
6
1    # l3
2
5
Task # l4
1
4    # l5
Task
3    # l6
2    # l7
1    # l8
Task # l9
{{< /highlight >}}

## 魔法时刻

借助 generator 外加一个我们自己实现的调度器，来解决 socket 编程中调用阻塞的问题。

{{< highlight diff "linenos=table">}}
--- a/aserver.py
+++ b/aserver.py
@@ -3,7 +3,26 @@

 from socket import *
 from fib import fib
-
+from collections import deque
+
+tasks = deque()
+recv_wait = { } # Mapping sockets -> tasks (generators)
+send_wait = { }
+
+def run():
+    while tasks:
+        task = tasks.popleft()
+        try:
+            why, what = next(task)  # Run to the yield
+            if why == 'recv':
+                # Must go wait somewhere
+                recv_wait[what] = task
+            elif why == 'send':
+                send_wait[what] = task
+            else:
+                raise RuntimeError("ARG!")
+        except StopIteration:
+            print("task done")

 def fib_server(address):
     #sock = socket(AF_INET, SOCK_STREAM)
@@ -12,22 +31,25 @@ def fib_server(address):
     sock.bind(address)
     sock.listen(5)
     while True:
+        yield 'recv', sock
         client, addr = sock.accept()  # blocking
         print("Connected ", addr)
-        fib_handler(client)
+        tasks.append(fib_handler(client))


 def fib_handler(client):
     while True:
+        yield 'recv', client
         req = client.recv(100)    # blocking
         if not req:
             break
         n = int(req)
         result = fib(n)
         resq = str(result).encode('ascii') + b'\n'
+        yield 'send', client
         client.send(resq)    # blocking
     print("Closed")


-fib_server(('', 25000))    
+tasks.append(fib_server(('', 25000)))
{{< /highlight >}}

从 14 到 27 行的 run 是真正的主函数，从 deque 的头部取出一个 task ，这个 task 的类型是 generator ，在它的 next 调用下将会把当前程序的执行暂停在这个 generator 的 yield 处——我们也是借此来实现公调度所有已知 socket 上的收/发阻塞调用。

第 35、44、51 行，在阻塞调用前插入 yield ，fib_server 和 fib_handler 都被变为了 generator。要注意的是 yield 返回了两个值: 值 1 是对**等待接收**和**等待发送**两个动作做出标记，值 2 是记录特定的 socket 对象。

最后，第 39、57 行是将原始 socket 程序中所有不同的 socket 连接加入到 deque 的末尾，成为新的轮循任务。

{{< highlight console >}}
$ python3 -i aserver.py
>>> tasks
deque([<generator object fib_server at 0x7fccb7c96468>])
>>> recv_wait
{}
>>> send_wait
{}
>>> run()
>>> tasks
deque([])
>>> recv_wait
{<socket.socket
    fd=3,
    family=AddressFamily.AF_INET,
    type=SocketKind.SOCK_STREAM,
    proto=0,
    laddr=('0.0.0.0', 25000)>:
<generator object fib_server at 0x7fccb7c96468>}

{{< /highlight >}}

进入 python 的交互模式，试运行当前的半成品。运行前 tasks 队列含有一个 socket 。开始运行后，它被放置到 recv_wait 中，等待一个新的 client 来连接。这里的 recv_wait 和 send_wait 被 David 比作冰球比赛中将球员发送到受罚席。

## 免线程版本

加上下面的 patch ，一个完全不借助线程的原型程序就可以工作了。

{{< highlight diff >}}
--- a/aserver.py
+++ b/aserver.py
@@ -4,13 +4,23 @@
 from socket import *
 from fib import fib
 from collections import deque
+from select import select

 tasks = deque()
 recv_wait = { }   # Mapping sockets -> tasks (generators)
 send_wait = { }

 def run():
-    while tasks:
+    while any([tasks, recv_wait, send_wait]):
+        while not tasks:
+            # No active task to run
+            # wait for I/O
+            can_recv,can_send,_ =select(recv_wait,send_wait,[])
+            for s in can_recv:
+                tasks.append(recv_wait.pop(s))
+            for s in can_send:
+                tasks.append(send_wait.pop(s))
+
         task = tasks.popleft()
         try:
             why, what = next(task)  # Run to the yield
@@ -52,4 +62,4 @@ def fib_handler(client):


 tasks.append(fib_server(('', 25000)))
-
+run()
{{< /highlight >}}

其中用到了 [select](https://docs.python.org/3/library/select.html) 模块，让它来代理检测哪些 socket 已处于准备好了的状态。

> 关于 IO 复用的更多细节，文末的参考文档中列出了一些博客和代码实例。

这样我们就可以将其重新加回到任务队列中，这样紧随其后的 next 调用也能将之前暂停的 generator 代码继续向下执行，直到下一个 yield 。

下一篇将继续讨论当前原型中存在的问题。

参考文档

> - [如何看懂冰球比赛？ “Hockey酱”的回答](https://www.zhihu.com/question/48783686)
> - [High-level I/O multiplexing example code](https://docs.python.org/3/library/selectors.html#examples)
> - 糖拌咸鱼 (2012-01-06) [Python网络编程中的select 和 poll I/O复用的简单使用](http://www.cnblogs.com/coser/archive/2012/01/06/2315216.html)
> - Vaidik Kapoor (2015-05-31) [Understanding Non Blocking I/O with Python - Part 1](https://vaidik.in/blog/understanding-non-blocking-io-with-python-part-1.html)

封面图片来自 [Pantone Color Picker](https://dribbble.com/shots/2511494-Pantone-Color-Picker) <a href="https://dribbble.com/marcoscv"><i class="fa fa-dribbble" aria-hidden="true"></i> Marcos Castro</a>
