+++
date = "2018-07-01T22:13:07+08:00"
title = "测试驱动开发"
showonlyimage = false
image = "/img/blog/head-first-cucumber/cucumber.png"
topImage =  "/img/blog/head-first-cucumber/cucumber.gif"
draft = false
weight = 1201
+++

使用[业务术语](https://en.wikipedia.org/wiki/Domain-specific_language)定义测试产品行为
<!--more-->

最近看到了阮一峰老师转发了一篇亚马逊 CTO Werner Vogels 2006 的文章 [Working Backwards](https://www.allthingsdistributed.com/2006/11/working_backwards.html), 提到亚马逊为了对用户的需求保持专注，其服务开发团队会采用一种“倒推”式的工作模式，可分为四步：

1. 先写新闻稿——用最简要的日常语言描述产品特性，外部世界该如何评价这个产品;
2. 再写常见问题(FAQ)文档——设想用户看到新闻稿或使用产品时提出的问题，做成一个问题列表作为新闻稿的补充，从用户视角思考这些问题答案；
3. 定义用户体验——尽可能详尽地描述用户地各种不同使用场景，要有模拟界面，要有代码片段;
4. 写用户手册——用户在使用时需要知道的一切，主要包含 **概念** 、**如何做**、**参考** 三部分，多类用户意味着多个手册；

经过这样一个步骤，文档在动手前就可用了，并且整个团队都对开发的产品有了相同的共识。

正好最近正在学习 [Cucumber](https://en.wikipedia.org/wiki/Cucumber_(software)) 其代表的[BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) 方法，和亚马逊的流程有些相似: 即防止仅从技术角度设计开发产品，强制团队紧紧围绕着业务逻辑开展工作。


参考文档

- Harsh Murari (2018-03-16) [BDD, Cucumber and Selenium WebDriver based Test Automation Framework in Python](https://medium.com/@hmurari/bdd-cucumber-and-selenium-webdriver-based-test-automation-framework-in-python-ae092a7581d3)
- Udemy Rahul Shetty (2018-03) [Cucumber with Java-Build Automation Framework in lesser code](https://www.udemy.com/share/100h30/)
- Udemy Karthik KK (2016-12) [Cucumber with Selenium Java (Basic)](https://www.udemy.com/share/1001OE/)
- Angie Jones (2016-08-24) [Rest-Assured with Cucumber: Using BDD for Web Services Automation](http://angiejones.tech/rest-assured-with-cucumber-using-bdd-for-web-services-automation/)
- GURU99 [Using Cucumber with Selenuim](https://www.guru99.com/using-cucumber-selenium.html)

https://www.tutorialspoint.com/cucumber/index.htm
https://docs.cucumber.io/
http://behave.readthedocs.io/en/latest/
https://www.guru99.com/smoke-sanity-testing.html

封面图片来自 [Summer-cucumber](https://dribbble.com/shots/2222787-Summer-cucumber) <a href="https://dribbble.com/xmushroomx"><i class="fa fa-dribbble" aria-hidden="true"></i> xMx</a>