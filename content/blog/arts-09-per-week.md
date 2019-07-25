+++
date = "2018-08-30T21:33:21+08:00"
title = "ARTS 2018w35"
showonlyimage = false
image = "/img/blog/arts-09-per-week/rooms_reveal.png"
topImage = "/img/blog/arts-09-per-week/rooms_reveal.gif"
draft = false
weight = 1835
tags = ["ARTS", "hugo"]
+++

在 Hugo 中配置应用多个主题
<!--more-->

## Algorithm

[19. Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/description/)

> Input: Given linked list: 1->2->3->4->5, and n = 2  
> Output: the linked list becomes 1->2->3->5

[StefanPochmann 的解答](https://leetcode.com/problems/remove-nth-node-from-end-of-list/discuss/8802/3-short-Python-solutions) 非常有意思： 

- 双指针：初始都指向 head，快指针先走 n 步，之后一起前移，直到最后一个元素，此时慢指针的下一个元素即应被移除的元素；
- 实现一个递归函数，计算各个元素的索引（从1计数，方向从头先尾): 对索引大于 n 的元素的值全部向尾部移动，这样返回的链表，并没有删除索引为 n 的元素左右的链表连接关系，“被删除”的永远是原链表的第一个元素；
- 实现一个递归函数，返回一个含两元素的 Tuple: (索引，剩余链表头部)，与第二种解法不同的是，这回是真正删除了索引为 n 的元素；

## Review 

Josh Dzielak (2018-07-06) [Harness the Power of Static Site Generators to Create Presentations](https://forestry.io/blog/harness-the-power-of-static-to-create-presentations/)

> - dzello [A Hugo theme for creating Reveal.js presentations](https://github.com/dzello/reveal-hugo)

一直想把两个 gohugo 的风格主题组合使用，一个用于日常博客，另一个用于单一题目的展示——比如 reveal.js 的 slides，这部分作为 TODO ，今年一定要完成。

> 拓展阅读

Forestry blogs related to [Hugo](https://forestry.io/categories/hugo/)

> - Régis Philibert (2018-04-13) [Build a JSON API With Hugo's Custom Output Formats](https://forestry.io/blog/build-a-json-api-with-hugo/)
> - Régis Philibert (2018-05-25) [Enhance Your Hugo JSON API Using Custom Output Formats and Netlify Redirects](https://forestry.io/blog/hugo-json-api-part-2/)
> - DJ Walker (2018-07-13) [Add Functionality to Your Hugo Site With Theme Components](https://forestry.io/blog/add-functionality-to-your-hugo-site-with-theme-components/)
> - DJ Walker (2018-08-17) [Keeping Content DRY: Data Relationships In Hugo](https://forestry.io/blog/data-relationships-in-hugo/)


## Tip

[yapf](https://github.com/google/yapf) 由 Google 出品，和 [autopep8](https://github.com/hhatto/autopep8#features) 不同，yapf 原则上无需人为修改，将输入代码直接格式化成一致的风格。这两个工具外加 [black](https://github.com/ambv/black) 的格式化实例比较，可以阅读： 

> - Kevin Peters (2018-06-03) [Auto formatters for Python👨‍💻🤖](https://medium.com/3yourmind/auto-formatters-for-python-8925065f9505)

为了防止自己忘记使用这些工具，最好将其加入 git 的 pre-commit hook，yapf 提供了对应的脚本，[见 README](https://github.com/google/yapf/tree/master/plugins#git-pre-commit-hook)。这样你在运行 `git commit` 后，yapf 会对这次提交的文件做格式化，你可以确认(`git add`) yapf 工具所做的更改或者为你需要保留的特例语句加上 ` # yapf: disable` 忽略格式化的规则。

## Share

[《最后的演讲》](https://en.wikipedia.org/wiki/The_Last_Lecture) 共同作者，也是书中主角的 Randy Pausch 是卡耐基梅隆大学计算机科技、人机交互及设计学科的教授，书中没有提及什么计算机知识，而是从他确诊癌症，决定去做最后演讲的故事讲起：回忆了他幼年时的教育和经历，成年后曾经遇到的那些挫折，如何“幸运”地实现了自己的理想，如何追求妻子，学术和职业生涯中又从哪里获得了重要的帮助，如何薪火相传的帮助他的学生，以及他对生活的建议。

他希望留给他的三个孩子，妻子和这个世界一些和他相关的东西。

> - （当我姐姐告诫她的孩子们擦干净鞋子，别弄脏弄乱兰迪舅舅的崭新的敞篷车时），我以“单身汉舅舅”的身份，认为：“就是这种警告才会让孩子失败。而且他们最后肯定会把我的车弄脏的，孩子们控制不住自己。”所以我把事情变简单了。当我姐姐在那儿一条条讲规矩时，我故意慢慢地打开一罐汽水，把罐子倒过来，把汽水都倒在了敞篷车后座的布椅座上。我借此传达了一个信息：人比东西更重要。……孩子们和我在一起时只有两条规矩：

> 1. 不许哭哭啼啼。
> 2. 无论我们一起做什么，都不能告诉妈妈。

> 在兰迪追求妻子的故事中，他的原则是“……围墙之所以存在，就是为了拦住愿望不够强烈的人，拦住那些无关人等。”故事最后他当然追到了杰伊，兰迪半炫耀地再次总结道：“围墙之所以存在是有原因的，就是为了给我们机会，展现自己的愿望有多强烈。

> 我一向认为踏实的人胜过时髦的人，因为时髦是短暂的，踏实才是长久的。

> 太多人一辈子都在不停抱怨。我一直认为，如果你把抱怨的精力拿出十分之一来解决问题，事情的结果就会超乎你的想象。

> 我发现，不少人每天都要花很多时间，担心别人对他们的看法。如果不那么在意别人在想什么，我们的生活和工作效率会提高三分之一。

> 一些陈词滥调，但很重要：
> - 如果一开始你没有成功……试试，再试试。
> - 和带你去舞会的人跳舞。
> - 幸运就是当机会遇到有准备的人。
> - 你能不能做到，取决于自己的想法。
> - 除此之外，林肯夫人，那场戏怎么样？——这是假设林肯在看戏时遇刺，某人对林肯夫人说的一句话。这是提醒不要关注一些不重要的细枝末节，忽略重要的整体。

> 在我看来，如果你比别人工作的时间更长，就能利用这段时间更加精通你的专业。这能让你更加高效，更加能干，甚至更加开心。努力工作就像是银行里的复利，回报会越来越多。工作之外，人生也是如此。成年后，我喜欢问那些老夫老妻的相处秘诀。他们都对我说了同样的话：“靠努力。”

> 做好准备的另一个方法，就是凡事往坏处想。诚然，我是个特别乐观的人，但是当我做决定的时候，总是会设想最坏的情况。……无论出了什么乱子，你都得有应急计划，这样你才能当一个乐观主义者。

> 在这个国家，我们的关注点往往放在人有哪些权利之上，这样也没错，但只谈权利不谈责任就没有任何意义了。

> 在我看来，父母的工作是鼓励孩子培养对生活的兴趣，激励他们去追寻自己的梦想。我们所能提供的最大帮助，就是让他们培养一套追求梦想的方法。

> 当（兰迪和他的妻子最终得到癌症复发并扩散转移的消息）离开医生的办公室后，我想起夕阳西下时（两个人开长途车到医生所在的城市，并在前一天一起去游乐场），我在水上乐园的高速滑梯上对杰伊说过的话。“即使明天的扫描结果不好，”我对她说，“我也只想告诉你活着真好，今天和你一起在这里活着真好。无论检查结果如何，我们得知结果时，我都不会马上就死，不仅如此，我第二天也不会死，第三天也不会，第四天也不会。所以，此时此刻是特别美好的一天。我想告诉你今天我特别开心。”


封面图片来自 [Rooms - Reveal](https://dribbble.com/shots/5018377-Rooms-Reveal) <a href="https://dribbble.com/Webshocker"><i class="fa fa-dribbble" aria-hidden="true"></i> Matjaz Valentar</a>