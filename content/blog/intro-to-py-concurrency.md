+++
date = "2017-09-08T23:27:52+08:00"
title = "python 并发简介"
showonlyimage = false
image = "/img/blog/intro-to-py-concurrency/sorting_kit8-net.png"
draft = false
weight = 101
+++

观看大神现场震撼演示
<!--more-->

## 兔子数列

斐波那契数列的一个简陋实现可能是这个样子的：
{{< highlight python >}}
def fib(n):
    if n <= 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)
{{< /highlight >}}

{{< highlight python >}}
python3

from fib import fib

fib(1)

fib(10)

fib(20)

{{< /highlight >}}

随着求解的数列位置越来越靠后，运算的时间也会指数级加长。

## 微服务版本

{{< highlight python >}}
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

{{< highlight python >}}

tty1 python3 server.py

tty2 nc localhost 25000 # telnet

{{< /highlight >}}

封面图片来自 [Sorting factory](https://dribbble.com/shots/3326497-Sorting-factory) <a href="https://dribbble.com/Frizler"><i class="fa fa-dribbble" aria-hidden="true"></i> Anton Fritsler (kit8)</a>
