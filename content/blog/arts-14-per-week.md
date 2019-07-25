+++
date = "2018-09-15T07:30:04+08:00"
title = "ARTS 2018w40"
showonlyimage = false
image = "/img/blog/arts-14-per-week/impossible.png"
topImage = "/img/blog/arts-14-per-week/impossible.gif"
draft = true
weight = 1840
tags = ["ARTS"]
+++

奇迹缔造者——致敬海伦·凯勒的老师——安·沙利文
<!--more-->

## Algorithm


## Review 

{{< figure src="/img/blog/arts-14-per-week/04-Replication_and_other_controllers_deploying_managed_pods.jpg" title="Replication and other controllers deploying managed pods" >}}

## Tip

当需要退订邮件列表时，一般用原订阅邮箱发一封特定标题的邮件到邮件列表就行了。但如果遇到公司改名、被并购等偶然事件发生后，邮箱后缀很可能发生变化，如此一来我们就同时有两个邮箱：新旧两个地址都能收信，但向外发信时邮箱服务器会强制用户通过新地址发出。这样上面说的邮件列表的自动退订就可能不工作了: 比如 Google Groups 需要通过 Web 页面的方式退订。

另一个问题是如果需要批量处理收件箱中来自某个邮件列表的邮件时，因为其发件人通常是`mail@list on behalf of someone@somewhere`，因此不能用简单的 `from: mail@list` 来过滤。解决方法是在 Outlook 中建一个基于邮件头部信息的关键字的规则过滤后处理。具体方法是打开任意一封信，查看文件属性，可以一个叫 Internet Header 的字段，内含很多字段可以规则的关键字过滤条件，比如 `List-ID: mail@list` ，然后定义相应处理动作即可。

## Share

最近读到一本小学生读物（非英文原文，被中文意译且带汉语拼音)——海伦·凯勒的散文集《假如给我三天光明》。读了几页，因为实在想象不出一个盲聋人如何能学会5种语言，完成学业，然后在全世界旅行讲演，于是我到网上找来一部电影[《奇迹缔造者》](https://movie.douban.com/subject/1305948/)——影片将老师 [Anne Sullivan](https://en.wikipedia.org/wiki/Anne_Sullivan) 对 [Helen Keller](https://en.wikipedia.org/wiki/Helen_Keller) 启蒙教育的最关键时期做了戏剧化的直白呈现——非常好的电影，基本解释了盲聋人学习语言的方式，过程好艰辛，令人泪奔。

当然最牛的一定是海伦的老师安妮，没有她的坚持和耐心，就不可能有海伦。而安妮在影片中提到了语言对人的重要作用令我印象深刻——因为联想起之前“得到”听书栏目出过一个纪念 [Steven Pinker](https://en.wikipedia.org/wiki/Steven_Pinker) 的书单:

- [The Language Instinct](https://en.wikipedia.org/wiki/The_Language_Instinct) (1994)

{{< figure src="/img/blog/arts-14-per-week/The-Language-Instinct.png" title="得到 APP <<语言本能>>" >}}

- [How the Mind Works](https://en.wikipedia.org/wiki/How_the_Mind_Works) (1997)

{{< figure src="/img/blog/arts-14-per-week/How-the-Mind-Works.png" title="得到 APP <<心智探索>>" >}}

- [The Blank Slate: The Modern Denial of Human Nature](https://en.wikipedia.org/wiki/The_Blank_Slate) (2002)

{{< figure src="/img/blog/arts-14-per-week/The-Blank-Slate.png" title="得到 APP 白板: <<科学和常识所揭示的人性奥秘>>" >}}

- [The Stuff of Thought: Language As a Window Into Human Nature](https://en.wikipedia.org/wiki/The_Stuff_of_Thought) (2007)

{{< figure src="/img/blog/arts-14-per-week/The-Stuff-of-Thought.png" title="得到 APP <<思想本质>>" >}}

这里直接借用“得到”的解读版的脑图。

另外史迪芬·平克的介绍如何进行英文写作的书[The Sense of Style](https://en.wikipedia.org/wiki/The_Sense_of_Style) 也应该非常值得一读。

## 拓展阅读

> - [Steven Pinker's TED talks](https://www.ted.com/speakers/steven_pinker)
> - [American manual alphabet](https://en.wikipedia.org/wiki/American_manual_alphabet)
> - [Three Days to See](http://www.afb.org/info/about-us/helen-keller/books-essays-and-speeches/on-the-senses/three-days-to-see-as-published-in-atlantic-monthly-january-1933/12345) by Helen Keller Jan, 1933

封面图片来自 [It's not so impossible](https://dribbble.com/shots/2452666-It-s-not-so-impossible) <a href="https://dribbble.com/brentclouse"><i class="fa fa-dribbble" aria-hidden="true"></i> Brent Clouse</a>