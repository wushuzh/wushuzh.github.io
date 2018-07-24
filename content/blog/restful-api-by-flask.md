+++
date = "2017-10-30T11:24:09+08:00"
title = "RestFul API"
showonlyimage = false
image = "/img/blog/restful-api-by-flask/rest.png"
topImage =  "/img/blog/restful-api-by-flask/rest.gif"
draft = false
weight = 106
+++

当下还在流行使用的远程通信方式
<!--more-->

本文是对 Bruno Krebs (2017-09-28) [Developing RESTful APIs with Python and Flask](https://auth0.com/blog/developing-restful-apis-with-python-and-flask/) 的实践版。其基本步骤是

- 搭建环境和基础代码
- 定义 RESTful API 
- 和 python 类映射
- 容器化

### 准备环境

步骤如下:

- 用 pipenv 创建一个 py3 含 flask 的虚拟环境
- cashman 模块化: init 和 最简 index.py 
- 创建启动脚本 bootstrap.sh 用于启动上述应用
- 创建 git 仓库提交初始版本


### 原型实验

目标是管理收支，首先是处理收入的两个 API 。不同的仅是动词: GET 和 POST，路经一样: 都是 incomes

{{< highlight console >}}
$ export no_proxy=localhost
$ curl http://localhost:5000/incomes
[
  {
    "amount": 5000, 
    "description": "salary"
  }
]
$ curl -X POST -H "Content-Type: application/json" -d '{
>    "description": "lottery",
>    "amount": 1000.0
> }' http://localhost:5000/incomes

$ curl http://localhost:5000/incomes
[
  {
    "amount": 5000, 
    "description": "salary"
  }, 
  {
    "amount": 1000.0, 
    "description": "lottery"
  }
]
{{< /highlight >}}

### 序列和对象的转换

步骤如下:

- cashman 模块下创建子模块 model
- 安装 marshmallow 
- 创建基类 Transaction 及 TransactionSchema 后者用于 python 和 json 的相互转化
- 创建子类 Income 和 Expense 和它们的 Schema 类，并装饰为在反序列化时实例化为 python 对象
- 在 index 中整合上述对象，分别添加 incomes 和 expenses 的 GET 和 POST 处理

尤其是子类中使用了 python 的内置函数 super，详见 

- [py3 stddoc super](https://docs.python.org/3.6/library/functions.html#super) 
- Raymond Hettinger (2011-05-26) [Python’s super() considered super!](https://rhettinger.wordpress.com/2011/05/26/super-considered-super/) 
- Stackoverflow 上 Aaron Hall 的回答 [Understanding Python super() with init methods](https://stackoverflow.com/a/27134600/4393386) [What does 'super' do in Python?](https://stackoverflow.com/a/33469090/4393386)

{{< highlight console >}}
$ ./bootstrap.sh &
$  * Serving Flask app "cashman.index"
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)

$ curl -X POST -H "Content-Type: application/json" -d '{
    "amount": 300.0,
    "description": "loan payment"
}' http://localhost:5000/incomes
127.0.0.1 - - [31/Oct/2017 11:40:22] "POST /incomes HTTP/1.1" 204 -
$ curl -X POST -H "Content-Type: application/json" -d '{
    "amount": 20,
    "description": "lottery ticket"
}' http://localhost:5000/expenses
127.0.0.1 - - [31/Oct/2017 11:40:34] "POST /expenses HTTP/1.1" 204 -
$ curl http://localhost:5000/expenses
127.0.0.1 - - [31/Oct/2017 11:40:43] "GET /expenses HTTP/1.1" 200 -
[
  {
    "amount": -50.0, 
    "created_at": "2017-10-31T11:40:13.333592", 
    "description": "pizza", 
    "typ": "TransactionType.EXPENSE"
  }, 
  {
    "amount": -100.0, 
    "created_at": "2017-10-31T11:40:13.333596", 
    "description": "Rock Concert", 
    "typ": "TransactionType.EXPENSE"
  }, 
  {
    "amount": -20.0, 
    "created_at": "2017-10-31T11:40:34.306376", 
    "description": "lottery ticket", 
    "typ": "TransactionType.EXPENSE"
  }
]
$ curl http://localhost:5000/incomes
127.0.0.1 - - [31/Oct/2017 11:40:49] "GET /incomes HTTP/1.1" 200 -
[
  {
    "amount": 5000.0, 
    "created_at": "2017-10-31T11:40:13.333573", 
    "description": "Salary", 
    "typ": "TransactionType.INCOME"
  }, 
  {
    "amount": 200.0, 
    "created_at": "2017-10-31T11:40:13.333588", 
    "description": "Dividends", 
    "typ": "TransactionType.INCOME"
  }, 
  {
    "amount": 300.0, 
    "created_at": "2017-10-31T11:40:22.923553", 
    "description": "loan payment", 
    "typ": "TransactionType.INCOME"
  }
]

{{< /highlight >}}

### 容器封装

这里使用是基于 [Alpine](https://en.wikipedia.org/wiki/Alpine_Linux) 的镜像，特点是精简，大小仅 80M+，而一般的 python3 镜像则 680M+。并作了安全、网络等方面的增强。

> 值得注意的是 Alpine 使用自己的包管理工具 apk，而其代理的设置根据这个 issus [#171 Running apk commands behind proxy fails](https://github.com/gliderlabs/docker-alpine/issues/171) 要写作完整形式。

和原文不同，我依然借助 docker-machine 做 image 的编译和运行，但遇到了报错。docker 的查错依然是通过 logs 命令查看日志，然后可以重启并改写 entrypoint 再进入交互模式。更多方法可以参考 Mark Betz (2016-03-24) [Ten tips for debugging Docker containers](https://medium.com/@betz.mark/ten-tips-for-debugging-docker-containers-cde4da841a1d)

{{< highlight console >}}
$ eval $(docker-machine env --no-proxy dev)
$ docker build -t cashman \
    --build-arg http_proxy=http://ip:port \
    --build-arg https_proxy=http://ip:port 
    .

$ docker run --name cashman \
     -d -p 5000:5000 \
     cashman
<hashid>

$ docker ps
$ docker ps -a
$ docker logs <hashid>
standard_init_linux.go:195: exec user process caused "no such file or directory"

$ docker run --name cashman -it --entrypoint /bin/sh cashman
/usr/src/app # ./bootstrap.sh &
/usr/src/app # /bin/sh: ./bootstrap.sh: not found
/usr/src/app # ls -l /bin/bash
ls: /bin/bash: No such file or directory
/usr/src/app # ls -l /bin/sh
lrwxrwxrwx    1 root     root            12 Oct 25 22:05 /bin/sh -> /bin/busybox

# modify bootstrap shebang to /bin/sh and rebuilt rerun and check
$ curl http://$(docker-machine ip dev):5000/incomes
[
  {
    "amount": 5000.0, 
    "created_at": "2017-10-31T08:39:52.342714", 
    "description": "Salary", 
    "typ": "TransactionType.INCOME"
  }, 
  {
    "amount": 200.0, 
    "created_at": "2017-10-31T08:39:52.342722", 
    "description": "Dividends", 
    "typ": "TransactionType.INCOME"
  }
]

{{< /highlight >}}

阮一峰 (2017-10-29 tweet): 远程通信的三代方法

  - 过去：SOAP ( HTTP + XML )
  - 现在：REST ( HTTP + JSON )
  - 未来：gRPC +  Protocol Buffers

  [Google’s gRPC: A Lean and Mean Communication Protocol for Microservices](https://thenewstack.io/grpc-lean-mean-communication-protocol-microservices/)

参考文档

> - [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
> - [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
> - [Why marshmallow?](https://marshmallow.readthedocs.io/en/latest/why.html)
> - [install marshmallow with flask failed with dep not resolved](https://github.com/kennethreitz/pipenv/issues/992)

封面图片来自 [Rest in the pool](https://dribbble.com/shots/3389841-Rest-in-the-pool) <a href="https://dribbble.com/mihkonyev"><i class="fa fa-dribbble" aria-hidden="true"></i> Mihail Koniaiev</a>
