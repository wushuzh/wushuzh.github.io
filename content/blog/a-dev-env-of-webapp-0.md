+++
date = "2018-05-17T21:17:18+08:00"
title = "Flask 开发 0"
showonlyimage = false
image = "/img/blog/a-dev-env-of-webapp-0/flask-do.png"
draft = false
weight = 601
tags = ["flask"]
+++

按官方教程，用TDD的方式写一个迷你博客
<!--more-->

Flask 的[官方教程](http://flask.pocoo.org/docs/1.0/tutorial/): 一个包含用户注册和登录、帖子的创建、修改、删除功能的迷你博客被包含在了[其源代码仓库中](https://github.com/pallets/flask/tree/1.0.2/examples/tutorial)。

- [项目结构](http://flask.pocoo.org/docs/1.0/tutorial/layout/): 和原教程稍微不同的是，用 pipenv 来管理虚拟环境，可以省去 venv 目录，仅有如下两个子目录: flaskr 和 tests 分别放置项目和测试代码;
- [创建应用](http://flask.pocoo.org/docs/1.0/tutorial/factory/): 一个 Flask 类的实例即为我们要构建的应用程序，为了避免全局变量，教程封装了一个 application factory 函数。用于完成所有和应用实例相关的配置、组件注册等初始化操作;
- [创建库表](http://flask.pocoo.org/docs/1.0/tutorial/database/): 定义所需的数据库表 DDL，关联到一个自定义指令上 init-db ，最后注册到应用实例，方便命令行使用;
- [组织视图](http://flask.pocoo.org/docs/1.0/tutorial/views/): View 函数内部是对请求的应答逻辑。而功能相关的一组视图可以定义在一个 Blueprint 内。本节中将身份验证作为一个 Blueprint 的组件注册至应用，包含了注册、登录、退出等功能;



参考文档

> - David Lord (2018-04-26) [Flask 1.0 Released](https://www.palletsprojects.com/blog/flask-1-0-released/)
> - [Flaskr: Intro to Flask, Test-Driven Development(TDD), and JavaScript](https://github.com/mjhea0/flaskr-tdd)

封面图片来自 [Flask](https://dribbble.com/shots/2018912-Flask) <a href="https://dribbble.com/amostov"><i class="fa fa-dribbble" aria-hidden="true"></i> Alex S. Mostov</a>
