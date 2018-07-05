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

Cucumber 本质是一个测试框架，它推荐使用者采用 BDD 的开发模式。即鼓励业务或架构人员和开发、测试三方在初始阶段多做简单高效的沟通，并输出一个用于描述系统功能的文档(.feature)，主要特点是

- 结合 Gherkin language 关键字 (feature, scenario) 用平实语言定义描述系统行为
- 按“准备条件——触发操作——检验行为”(Given, When, Then)的格式围绕一个特性做不同场景下的预期行为描述，每句描述都叫做Step
- 支持对测试场景的标签标记，比如按 JIRAID 标记，或按 [Sanity 或 Smoke](https://www.guru99.com/smoke-sanity-testing.html) 不同类别标记
- 支持对某个测试场景的大纲化，以及对不同参数化代入后的反复执行

{{< figure src="/img/blog/head-first-cucumber/feature.png" title="Feature file" >}}

之后的开发会围绕着上述 feature 文件(也可以理解为传统的 test suite) 进行开发，Junit 的 Test Runner 可以用来调用 cucumber , 第一次 cucumber 能产生和之前 Gherkin Step 对应的空方法，这部分叫做 StepDefinition 。这一对应的过程叫做 glue ，即将业务术语和编程语言做粘合和链接:

- 用各种库做预置条件的模拟和断言检测
- 实现 Hooks 以完成测试前的准备和测试后的清理
- 可以用不同包、类将不同类别的 StepDefinition 分别组织管理

{{< figure src="/img/blog/head-first-cucumber/stepDefinition.png" title="Step Definition class" >}}

更加细节的信息可以查看[官方文档](https://docs.cucumber.io/installation/) 以及 Tutorialpoint [Cucumber tutorial](https://www.tutorialspoint.com/cucumber/index.htm) 内容挺全，但文档格式尤其是显示 feature 文件的排版时经常错误。

后续我会继续学习一下 cucumber 和其他测试库([Selenium](https://en.wikipedia.org/wiki/Selenium_(software))、[Rest Assured](http://rest-assured.io/)）和其他框架 [Spring](https://thepracticaldeveloper.com/2018/03/31/cucumber-tests-spring-boot-dependency-injection/) 结合使用的相关技巧。

<img alt="End Picture" src="/img/blog/head-first-cucumber/cucumber_gherkin.jpg" class="img-responsive">

[Made in Manhattan](https://dribbble.com/shots/3560566-Made-in-Manhattan-eaten-in-London)
 <a href="https://dribbble.com/andrew-nye"><i class="fa fa-dribbble" aria-hidden="true"></i> Andrew Nye</a>





参考文档

- Harsh Murari (2018-03-16) [BDD, Cucumber and Selenium WebDriver based Test Automation Framework in Python](https://medium.com/@hmurari/bdd-cucumber-and-selenium-webdriver-based-test-automation-framework-in-python-ae092a7581d3)
- Udemy Rahul Shetty (2018-03) [Cucumber with Java-Build Automation Framework in lesser code](https://www.udemy.com/share/100h30/)
- Udemy Karthik KK (2016-12) [Cucumber with Selenium Java (Basic)](https://www.udemy.com/share/1001OE/)
- Angie Jones (2016-08-24) [Rest-Assured with Cucumber: Using BDD for Web Services Automation](http://angiejones.tech/rest-assured-with-cucumber-using-bdd-for-web-services-automation/)
- GURU99 [Using Cucumber with Selenuim](https://www.guru99.com/using-cucumber-selenium.html)


封面图片来自 [Summer-cucumber](https://dribbble.com/shots/2222787-Summer-cucumber) <a href="https://dribbble.com/xmushroomx"><i class="fa fa-dribbble" aria-hidden="true"></i> xMx</a>