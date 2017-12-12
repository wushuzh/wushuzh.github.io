+++
date = "2017-12-01T21:27:56+08:00"
title = "Flask 开发 4"
showonlyimage = false
image = "/img/blog/a-dev-env-of-webapp-4/tori-gate.png"
draft = false
weight = 602
+++

集成容器数据库，为应用创建测试集
<!--more-->

本系列是 [Microservices with Docker, Flask, and React](https://testdriven.io/) 我的实践版——只要能达到相同效果，我会尝试替换更新原教程中的工具，升级版本，并简述填过或待填的坑。

### 模板

### No users

### All users

### add users

{{< highlight console >}}
$ docker-compose run users-service flask test
Starting users-db ... done
...
test_main_no_users (test_users.TestUserService)
Ensure the main route behaves correctly ... ok
test_main_with_users (test_users.TestUserService)
Ensure the main route behaves correctly ... ok
test_main_add_user (test_users.TestUserService)
Ensure a new user can be added to the database. ... ok

----------------------------------------------------------------------
Ran 15 tests in 3.879s

OK

{{< /highlight >}}

### split as main and app

### Coverage

### Nginx 


参考文档

> - 

封面图片来自 [Torii Gate](https://dribbble.com/shots/2842757-Torii-Gate) <a href="https://dribbble.com/vijairamalingam"><i class="fa fa-dribbble" aria-hidden="true"></i> Vijayakumar Ramalingam</a>  
