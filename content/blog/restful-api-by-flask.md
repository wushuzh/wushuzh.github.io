+++
date = "2017-10-30T11:24:09+08:00"
title = "RestFul API 开发"
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
- cashman 模块化: 空__init__.py 和 最简 index.py 
- 创建启动脚本 bootstrap.sh 用于启动上述应用
- 创建 git 仓库提交初始版本


### RESTful Endpoint

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

### 映射到 Python 类

步骤如下:

- cashman 模块下创建子模块 model
- 安装 marshmallow 
- 创建基类 Transaction 及 TransactionSchema 转化 Transcation 实例和 json 对象


<details>
  <summary>阮一峰: 远程通信的三代方法</summary>
    {{< tweet 924517735832285184 >}}
</details>

参考文档

> - [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
> - [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
> - [Why marshmallow?](https://marshmallow.readthedocs.io/en/latest/why.html)
> - [install marshmallow with flask failed with dep not resolved](https://github.com/kennethreitz/pipenv/issues/992)

封面图片来自 [Rest in the pool](https://dribbble.com/shots/3389841-Rest-in-the-pool) <a href="https://dribbble.com/mihkonyev"><i class="fa fa-dribbble" aria-hidden="true"></i> Mihail Koniaiev</a>
