+++
date = "2017-06-28T19:18:51+08:00"
title = "像雨果一样写作 1"
showonlyimage = false
image = "/img/blog/blog-using-hugo-1/desktop.png"
draft = false
weight = 20
tags = [ "Blogging", "Hugo", "GitHub" ]
categories = [ "Writing" ]
series = [ "Blog like a Pro" ]
+++

记录下如何用 GitHub 和 Hugo 搭建博客。
<!--more-->

## 为什么要自行搭建

写文章当下的首选依然

1. 开通微信公众号
2. 微博的头条文章
3. 甚至知乎专栏、头条等等

但考虑到控制权和便捷修改已发布文章等因素，个人搭建则更具优势。关于这方面更为深入的思考，见文末[左耳老师的文章](http://coolshell.cn/articles/17391.html)。

<img alt="do it yourself" src="/img/blog/blog-using-hugo-1/diy.jpg" class="img-responsive">

像博客这种类型网站的搭建，推荐使用静态站点生成器( 也许你听说过 Jekyll 或其他类似工具 )来生成站点。这么做当然需要折腾一番，但想想你可以得到的：

- 随时选择、微调、更换主题的渲染风格
- 能使用 Markdown 写作
- 能定制集成各类工具提高效率
- 能使用版本控制，查看，比较，更改已发布的文章

## 选择文具

取决于你选用钢笔，毛笔还是录音笔来码字，耗费的时间，最终成文风格，都会有很大差异，因此谨慎地选择一款适合自己的静态站点生成器非常必要。我是知道了[师兄](http://air.googol.im/)用 Hugo ，我才跟风的：一方面避免自己在选择性障碍的泥潭中挣扎，另一方面就算遇到什么诡异问题，总算是可以求教。

和 Jekyll 用 RubyGem 管理依赖不同， Hugo 因为是用 Go 语言开发，几乎所有依赖都打入其二进制执行程序中，相比之下其安装就显得非常简单——其实压根就没有安装，直接解压就完了。大家可能都有这样的经历：刚有点激情撸码，立刻开始准备软件环境，等折腾一通终于把安装和依赖都搞定，之前的激情也正好耗光，所以干净利落的安装真的非常重要。

Huge 使用步骤和 Jekyll 差不多：相似的创建站点命令，相似的全局配置文件，相似的扉页元数据设定，以及都用 Markdown 编写文章。

这些初始配置完成后是设定“稿纸”样式。与 [Jekyll](http://jekyllthemes.org/) 不同， Hugo 不提供默认的主题，所以你需要尽早从其[主题画廊](https://themes.gohugo.io/)中选出中意的主题，克隆到新建站点的 themes 子目录中，后续当然也可以自由切换主题，定义各种细节。

## 选出版社

除了在自己的 VPS 上发布。另一个选择就是 GitHub Pages 。虽说 GitHub 去年迫于资本压力，对此服务做了一些[限制](https://help.github.com/articles/what-is-github-pages/#usage-limits)( 比如最大带宽使用量每月 100 GB 或 100k 请求 )，但对于普通人来说绝对够用了。

具体步骤不再敷述，基本步骤包括

1. 新建仓库，命名为 username.github.io ，添加并设定 br:sources 为默认分支
2. 克隆仓库至本地，hugo 建站，并克隆心怡的主题
3. 一方面令分支 sources 忽略 public 子目录，同时让分支 master 仅包含 public 子目录
4. 添加开篇博客 Hello World ，在[本地](localhost:1313)确认后将 public [部署](https://hjdskes.github.io/blog/update-deploying-hugo-on-personal-gh-pages/)到 GitHab ，将浏览器指向[远端](https://username.github.io)再次查看确认

上述步骤仅仅是个 dry-run ，所以尽量在最短时限内完成。千万不要纠结在各种细节上不能自拔。后面有的是微调优化的时间。

<img alt="editorial-story" src="/img/blog/blog-using-hugo-1/editor.jpg"  style="width:70%; height:70%; display:block; margin: auto;">

参考文档

> - Henry Jenkins (2008-04-07) [Why Academics Should Blog](http://henryjenkins.org/2008/04/why_academics_should_blog.html)
> - 刘未鹏，(2009-02-15) [为什么你应该（从现在开始就）写博客](
http://mindhacks.cn/2009/02/15/why-you-should-start-blogging-now/)
> - 阮一峰，(2006-12-22) [1. 为什么要写Blog](http://www.ruanyifeng.com/blog/2006/12/why_i_keep_blogging.html) (2010-04-29) [2. 与王建硕的对话](http://www.ruanyifeng.com/blog/2010/04/talk_with_wangjianshuo.html)
> - 陈素封，(2014-05-03) [为什么你要写博客](https://zhuanlan.zhihu.com/p/19743861)
> - 师北宸，(2015-11-27) [如何通过写作取得职场突围](http://shibeichen.com/post/134060649456)
> - ActionThinker, (2016-11-06) [我为什么要重新开始写博客](http://actionthinker.com/2016-11-06-why-i-reopen-my-blog/)
> - Vultr (2016-01-05) [How To Create A Blog With Hugo](https://www.vultr.com/docs/how-to-create-a-blog-with-hugo)
> - Justin E. (2015-11-09) [How To Install and Use Hugo, a Static Site Generator, on Ubuntu 14.04]( https://www.digitalocean.com/community/tutorials/how-to-install-and-use-hugo-a-static-site-generator-on-ubuntu-14-04)
> - [字母](http://www.zimustudio.com/hugo.html) 内含 Hugo 中文翻译，但我没看，留在这里仅作参考
