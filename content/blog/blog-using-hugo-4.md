+++
date = "2017-07-13T20:01:51+08:00"
title = "像雨果一样写作 4"
showonlyimage = false
image = "/img/blog/blog-using-hugo-4/discuss.png"
draft = false
weight = 23
tags = [ "Blogging", "Hugo", "Disqus", "Comment", "Analyze" ]
categories = [ "Writing" ]
series = [ "Blog like a Pro" ]
+++

如何开放评论，以及知晓谁在访问我的网站
<!--more-->

## 获取反馈

以下几种情况可能获得反馈：

1. 读者觉得文章特别有用，忍不住点个赞，或推荐到推特；
2. 读者希望就特定情境，做更为深入的交流讨论；
3. 读者发现了行文中错误，拍砖做个友情提示；
4. 无关人等发广告贴或水军的恶意评论; （据说 Disqus 就可以作为访问门槛，挡住无关人员的评论）

无论如何，上述这种提交评论的 POST 操作是没法在静态网站上完成的，更何况还要对读者做身份标识，在功能和健壮之间做出平衡，仅不因为点个赞，做一轮注册操作，又不会因为缺少身份识别，让广告满天飞 。

<img alt="Disqus Comment" src="/img/blog/blog-using-hugo-4/disqus_comments.png" class="img-responsive">

于是就出现了像 [Disqus](https://en.wikipedia.org/wiki/Disqus) 这样的专业提供评论服务的厂商，[Disqus](https://disqus.com/) 不仅仅被大量的个人博客站点，就连不少大中型的新闻、娱乐类网站，如 CNN、The Daily Telegraph、IGN 也都是其用户。

作为一个第三方 Javascript 应用，通过用户浏览器，它将自己注入静态博客站点内，其内容都为动态生成，根据当前页面上的原数据（ID、URL、标题等），在隔离的 iframe 中和 Disqus 后台服务器交互数据。详情可看 Disqus forum 上的 [How Disqus Work ](https://help.disqus.com/customer/portal/articles/466187-how-does-disqus-work-) 和 Quora 上内部员工的简述 [How does Disqus work](https://www.quora.com/How-does-Disqus-work) 。

Hugo 的官网文档上展示了如何仅用一行代码通过预制 template 为自己的页面加上评论区。在这个基础上，参考下面两篇博客，采用了点击才展开评论区的加载方式。

> Amit A. (2014-08-25) [Load disqus comments on click](https://www.labnol.org/internet/load-disqus-comments-on-click/28653/)  
> Seth W. (2014-11-15) [Load disqus on demand](https://internet-inspired.com/wrote/load-disqus-on-demand/)  

关于评论显示样式的更新，参见[这篇博客](https://blog.jamespan.me/2015/04/18/goodbye-duoshuo/)，我先标记为 TODO 放在这里

<img style="width:70%; height:70%; display:block; margin: auto 10%;" alt="Free Speech" src="/img/blog/blog-using-hugo-4/free_speech.png" class="img-responsive">

## 量化统计

为了回答下面这些问题，我们就需要像 [Google Analytics](https://en.wikipedia.org/wiki/Google_Analytics) 追踪网站流量的工具了。

- 读者都来自哪里？用什么设备？
- 哪些文章被最多围观？
- 网站访问速度如何？

<img alt="Google Analytics" src="/img/blog/blog-using-hugo-4/ga_logo.png" class="img-responsive">

只要你能访问 Google ，注册服务并获取 GA Tracking ID 非常简单。之后回到 Hugo 上也只需一行[配置代码](https://gohugo.io/extras/analytics/)就可以让其生效。

简单调研了一下:  

- SEO shifu (2017-03) [Is Google Analytics Blocked in China?](https://chineseseoshifu.com/blog/is-google-analytics-blocked-in-china.html)
- Grizzly Panda (2017-05-10) [How to Use Google Analytics in China](http://grizzlypandamarketing.com/use-google-analytics-china/)
- Stackexchange (2015-09) 的问题解答 [How to remove Analytics' DoubleClick](https://webmasters.stackexchange.com/a/85194)

GA 应该是少数我们尚可部分使用的服务，但如果你想换别的，比如 [百度统计](https://tongji.baidu.com/web/welcome/login) ，可以参考 Quora 上的这篇 [Alternatives to GA in China](https://www.quora.com/What-are-some-alternatives-to-Google-Analytics-in-China)

关于 GA 分析管理，新手最简单的操作是：新建一个 view 把自己的访问流量屏蔽掉：  

- 无论是通过浏览器插件 ( [插件1](https://chrome.google.com/webstore/detail/block-yourself-from-analy/fadgflmigmogfionelcpalhohefbnehm) 或 [插件2](https://chrome.google.com/webstore/detail/google-analytics-opt-out/fllaojicojecljbmefodhfapmkghcbnh?hl=zh-CN) )  
- 创建过滤器 ( [参考1] (http://www.daniloaz.com/en/5-ways-to-exclude-your-own-visits-from-google-analytics/) 以及 [参考2](http://andrewmiguelez.com/exclude-yourself-from-google-analytics/) )

<img style="width:70%; height:70%; display:block; margin: auto 10%;" alt="Analytics" src="/img/blog/blog-using-hugo-4/seo-and-social-media.png" class="img-responsive">

缩略语解释

其他参考文档  

> Tony Bai (2015-09-23) [使用Hugo搭建静态站点](http://tonybai.com/2015/09/23/intro-of-gohugo/)  
> Jeff Liu (2016-01-01) [Part 10: Hugo with Google Analytics](http://usedgoodies.com/2016/01/01/setting-up-a-simple-blog-with-a-static-website-generator---part-10-hugo-with-google-analytics/)



文中图片来源网络
